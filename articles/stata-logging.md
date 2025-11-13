# Stata Log Files: Technical Reference

## Key Points

1. **Stata logs use persistent file streams** - File handles remain open throughout session, not opened/closed per write [1]
2. **30-60 minute network failures are SMB timeouts** - Default SMB/CIFS session timeouts cause file handle invalidation [2,3]  
3. **Use local logging + periodic copying for network shares** - Avoids timeout issues entirely
4. **Logs are buffered** - Output buffers periodically, use `qui log query` to force immediate writes
5. **Text format more reliable than SMCL** - `log using "file.log", text replace` creates smaller, portable files
6. **Implement retry mechanisms** - Wrap network operations in error-handling loops

## Overview

Stata log files capture commands and output during sessions. Understanding internal file handling is crucial for troubleshooting network share issues.

## Internal Mechanics

### File Handle Management
Stata uses **persistent file handles** - files stay open for the entire session:
- Opens file handle and maintains internal buffer
- Writes periodically based on buffer size/timing  
- Buffer flushes every few seconds, at capacity, before certain commands, or on close

### Stream-Based Operation
- Log files remain open with active handles until `log close`
- Can suspend/resume without closing (proves persistent handles)
- Network issues cause timeouts because handles stay open

## Log Formats

### SMCL vs Text
- **SMCL** (default): Binary format with formatting, requires Stata to read
- **Text**: Plain text, smaller files, universally readable
- **Recommendation**: Use text format for reliability: `log using "file.log", text replace`

### File Operations
```stata
log using "analysis.log", replace  // Overwrites existing
log using "analysis.log", append   // Adds to existing
```

## File Handling

### File Locking
- Exclusive lock placed on file during session
- File handle remains open until explicitly closed  
- Network file systems have different locking mechanisms

### Buffer Control
```stata
log using "mylog.log", replace
display "Important message"
qui log query  // Forces buffer flush
```

## Network Share Issues

### 30-60 Minute Timeout Pattern
**Root cause**: SMB/CIFS session timeouts (default 45-60 minutes)

### Technical Causes
1. **SMB Session Timeout** - Server closes "stale" file handles after inactivity
2. **Opportunistic Locks** - Server revokes file locks unexpectedly  
3. **Network Latency** - High latency causes buffer flush timeouts

### Architecture Problem
```
Local:  Stata ↔ File Handle ↔ Local Disk
Network: Stata ↔ File Handle ↔ Network ↔ Server ↔ Remote Disk
```
Multiple failure points in network path.

## Solutions

### Network Share Workaround
```stata
// Log locally first
log using "C:\temp\analysis.log", text replace
// Your analysis code here
log close
// Copy to network share
copy "C:\temp\analysis.log" "\\server\share\analysis.log", replace
```

### Robust Logging Wrapper
```stata
program define network_log
    args logname
    capture log close _all
    local templog "C:\temp\stata_temp.log"
    log using "`templog'", replace text
    // Analysis code here
    log close
    // Retry copy mechanism
    local attempts = 0
    while `attempts' < 5 {
        capture copy "`templog'" "`logname'", replace
        if _rc == 0 continue, break
        local attempts = `attempts' + 1
        sleep 5000
    }
end
```

### System Configuration
```batch
// Windows: Increase SMB timeout (requires admin)
net config server /autodisconnect:240  // 4 hours
```

### Best Practices
```stata
// Always specify format and close properly
capture log close _all
log using "analysis.log", text replace
// ... analysis ...
log close
```

## Troubleshooting

### Quick Diagnostics
```stata
log query                    // Check current log status
capture log close _all       // Close stuck logs
tempfile test; save "`test'", replace emptyok
copy "`test'" "\\server\share\test.dta", replace  // Test network access
```

### Common Errors
| Error | Solution |
|-------|----------|
| `file could not be opened` | Use local logging + copy |
| `log file already open` | `capture log close _all` |
| `disk full` | Check server space |
| `file not found` | Verify network path |

### Emergency Recovery
```stata
capture log close
log using "recovery_`c(current_time)'.log", text replace
display "Log session recovered"
```

## Summary

**Problem**: Stata's persistent file handles cause 30-60 minute network timeouts  
**Solution**: Log locally first, then copy to network shares  
**Best Practice**: Always use text format and explicit log management

## References

### Official Documentation
[1] StataCorp. (2023). *Stata Programming Reference Manual*. "file — Read and write ASCII text and binary files" 
[2] Microsoft Corporation. (2023). *SMB Protocol Documentation*. https://docs.microsoft.com/en-us/windows-server/storage/file-server/
[3] Samba Team. (2023). *SMB/CIFS Server Configuration*. https://www.samba.org/samba/docs/

### Resources  
- **Statalist**: Official Stata forum - https://www.statalist.org/
- **Stata Support**: https://www.stata.com/support/
- **UCLA Stata FAQ**: https://stats.idre.ucla.edu/stata/

### Tools
- **Wireshark**: Network debugging - https://www.wireshark.org/
- **Process Monitor**: Windows file activity monitoring
- **lsof/netstat**: Connection monitoring (Unix)