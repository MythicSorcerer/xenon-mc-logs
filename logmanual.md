# Minecraft Server Log Management - Usage Manual

## Overview

This manual covers two powerful scripts for managing and reading Minecraft server logs:
- `decompress_logs.sh` - Decompresses `.log.gz` files into readable format
- `readall.sh` - Displays all log files in chronological order

## Directory Structure

```
/opt/minecraft/server/logs/
├── decompress_logs.sh          # Decompression script
├── latest.log                  # Current server log
├── 2024-09-15-1.log.gz        # Compressed archived logs
├── 2024-09-14-1.log.gz
├── 2024-09-13-1.log.gz
└── readable/                   # Decompressed logs directory
    ├── readall.sh             # Log reading script
    ├── 2024-09-15-1.log       # Decompressed logs
    ├── 2024-09-14-1.log
    └── 2024-09-13-1.log
```

---

## Script 1: decompress_logs.sh

### Purpose
Automatically decompresses all `.log.gz` files in the logs directory and places readable versions in the `readable/` subdirectory.

### Location
`/opt/minecraft/server/logs/decompress_logs.sh`

### Basic Usage

```bash
cd /opt/minecraft/server/logs
./decompress_logs.sh
```

### Features

- **Smart Skipping**: Automatically skips files that have already been decompressed
- **Safe Operation**: Preserves original compressed files
- **Progress Tracking**: Shows which files are being processed
- **Error Handling**: Reports successful and failed operations
- **Auto-Organization**: Creates and manages the `readable/` directory

### Sample Output

```
=== Minecraft Log Decompression Utility ===
Source directory: /opt/minecraft/server/logs
Target directory: /opt/minecraft/server/logs/readable

Found 3 .gz file(s) to decompress:
  2024-09-15-1.log.gz
  2024-09-14-1.log.gz
  2024-09-13-1.log.gz

Processing: 2024-09-15-1.log.gz
  ✓ Decompressed: 2024-09-15-1.log (1234567 bytes)
Processing: 2024-09-14-1.log.gz
  Already exists, skipping: 2024-09-14-1.log
Processing: 2024-09-13-1.log.gz
  ✓ Decompressed: 2024-09-13-1.log (987654 bytes)

=== Summary ===
Successfully decompressed: 2 files
Total processed: 3 files
```

### Best Practices

- **Run regularly**: Execute after server restarts or daily to catch new archived logs
- **No cleanup needed**: The script handles everything automatically
- **Safe to re-run**: Skips existing files, so running multiple times is harmless

---

## Script 2: readall.sh

### Purpose
Displays all log files (decompressed + current) in chronological order from oldest to newest, with clear file headers.

### Location
`/opt/minecraft/server/logs/readable/readall.sh`

### Basic Usage

#### Standard Output (All at once)
```bash
cd /opt/minecraft/server/logs/readable
./readall.sh
```

#### Recommended Usage (With pager, starting at bottom)
```bash
./readall.sh | less +G
```

### Features

- **Chronological Order**: Sorts all logs from oldest to newest
- **Includes Current Log**: Automatically includes `latest.log` in proper chronological position
- **Clear Headers**: Shows filename, date, size, and path before each log
- **Size Information**: Displays total size and individual file sizes
- **Smart Confirmation**: Asks before displaying large amounts of text
- **Bottom-Start**: Recommended usage starts at newest logs (most relevant)

### Sample Output

```
=== Minecraft Log Reader ===
Reading logs from: /opt/minecraft/server/logs/readable

Found 4 log file(s) (ordered oldest to newest):
  2024-09-13-1.log (2024-09-13)
  2024-09-14-1.log (2024-09-14)
  2024-09-15-1.log (2024-09-15)
  latest.log (2024-09-15)

Total size of all logs: 4.2MB
This may produce a lot of output. Consider using: ./readall.sh | less +G

Continue and display all logs? (y/N): y

================================================================================
                           DISPLAYING ALL LOGS                                  
================================================================================

################################################################################
# FILE: 2024-09-13-1.log
# DATE: 2024-09-13 10:30:25
# SIZE: 1.1MB
# PATH: /opt/minecraft/server/logs/readable/2024-09-13-1.log
################################################################################

[10:30:25] [Server thread/INFO]: Starting minecraft server version 1.21
[10:30:25] [Server thread/INFO]: Loading properties
...
```

---

## Navigation with `less`

When using `./readall.sh | less +G` (recommended), you get these navigation options:

### Essential Commands
- **`G`** - Jump to bottom (newest logs)
- **`g`** - Jump to top (oldest logs)
- **`q`** - Quit and return to command line
- **`Space`** or **`Page Down`** - Scroll down (toward older logs)
- **`Page Up`** - Scroll up (toward newer logs)

### Search Functions
- **`/pattern`** - Search forward for "pattern"
- **`?pattern`** - Search backward for "pattern"
- **`n`** - Go to next search result
- **`N`** - Go to previous search result

### Useful Search Examples
```bash
# After running ./readall.sh | less +G, search for:
/ERROR          # Find error messages
/WARN           # Find warnings
/joined         # Find player joins
/left           # Find player disconnections
/FATAL          # Find critical errors
```

---

## Common Workflows

### Daily Log Review
```bash
# 1. Decompress any new archived logs
cd /opt/minecraft/server/logs
./decompress_logs.sh

# 2. Review recent activity (starts at newest)
cd readable
./readall.sh | less +G
```

### Troubleshooting Server Issues
```bash
# Quick check of recent logs
cd /opt/minecraft/server/logs/readable
./readall.sh | less +G

# Search for errors in the viewer:
# Press / then type: ERROR
# Press n to find next error
# Press N to find previous error
```

### Finding Specific Events
```bash
cd /opt/minecraft/server/logs/readable
./readall.sh | less +G

# Then search for specific patterns:
/player_name joined    # Find when a player joined
/Stopping server      # Find server shutdowns
/Starting server      # Find server startups
```

---

## File Management

### Automatic Cleanup
- The scripts don't delete any files automatically
- Original `.gz` files are preserved
- Decompressed files accumulate in `readable/`

### Manual Cleanup (if needed)
```bash
# Remove old decompressed logs (keeps originals)
cd /opt/minecraft/server/logs/readable
sudo rm -f *.log

# Then re-run decompress to get fresh copies
cd ..
./decompress_logs.sh
```

### Storage Considerations
- Decompressed logs take more space than compressed ones
- Monitor disk usage if you have many large log files
- Consider cleaning up very old decompressed logs periodically

---

## Troubleshooting

### "Permission denied" errors
```bash
# Fix script permissions
sudo chmod +x /opt/minecraft/server/logs/decompress_logs.sh
sudo chmod +x /opt/minecraft/server/logs/readable/readall.sh

# Fix ownership
sudo chown minecraft:mc /opt/minecraft/server/logs/decompress_logs.sh
sudo chown minecraft:mc /opt/minecraft/server/logs/readable/readall.sh
```

### "No .gz files found"
- This is normal if all logs have already been decompressed
- New `.gz` files appear when the server rotates logs (usually daily or on restart)

### "No log files found" 
- Run `decompress_logs.sh` first to create decompressed files
- Check that you're in the correct directory

### Large output warnings
- Always use `| less +G` for large log collections
- The scripts warn you about file sizes before displaying

---

## Tips and Best Practices

1. **Use the pager**: Always use `./readall.sh | less +G` for log review
2. **Regular decompression**: Run `decompress_logs.sh` daily or after server restarts
3. **Start from the bottom**: Most recent logs are usually most relevant
4. **Use search**: Learn the `less` search commands to quickly find specific events
5. **Monitor disk space**: Keep an eye on storage if you accumulate many large logs
6. **Backup important findings**: Copy relevant log sections when troubleshooting

---

## Quick Reference

| Task | Command |
|------|---------|
| Decompress new logs | `cd /opt/minecraft/server/logs && ./decompress_logs.sh` |
| Read all logs (newest first) | `cd readable && ./readall.sh \| less +G` |
| Search for errors | In `less`: `/ERROR` then `n` for next |
| Jump to newest logs | In `less`: `G` |
| Jump to oldest logs | In `less`: `g` |
| Quit log viewer | In `less`: `q` |

---

## Support

These scripts are designed to be self-explanatory with helpful output messages. If you encounter issues:

1. Check file permissions and ownership
2. Verify you're in the correct directory
3. Look at the error messages - they usually explain the problem
4. The scripts include safety checks and confirmation prompts for potentially destructive operations
