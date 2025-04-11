# macspector.sh — macOS Background Log Inspector

A menu-driven, terminal-based tool to investigate macOS background processes, memory pressure, and system daemons using the unified logging system.

## ✅ Perfect For:
- Diagnosing runaway Spotlight processes (`mds_stores`, `fseventsd`, `spotlightknowledged`)
- Investigating RAM/swap pressure and slowdowns
- Troubleshooting login and LaunchDaemon failures
- Detecting metadata import crashes or indexing loops
- Monitoring Time Machine or Backblaze behavior

## 🔧 Features
- Menu-based interactive CLI
- Targeted `log show --predicate` queries for:
  - Spotlight indexing and metadata importers
  - FSEvents (filesystem change events)
  - Swap & memory pressure (jetsam, vmpressure)
  - Launchd service restarts
  - APFS snapshot-related logs
  - Backblaze agent issues and memory usage
- Human-readable memory output via `ps aux`
- Timestamped full log capture saved to `/tmp/macos_diagnostics_<timestamp>.log`
- Optional "Run ALL" mode to batch execute all checks

## 🖥️ Usage
```bash
chmod +x macspector.sh
./macspector.sh         # defaults to 30m
./macspector.sh 1h      # show logs from the past 1 hour
```

## 📋 Menu Overview
```
h) Help
1)  Spotlight - spotlightknowledged
2)  MDS - mds_stores
3)  FSEvents - fseventsd
4)  Backup - backupd
5)  Swap and memory pressure
6)  LaunchDaemon failures
7)  Metadata worker crashes
8)  High memory warnings (jetsam/highwater)
9)  WindowServer issues
10) User Login & LoginWindow
11) APFS Snapshot logs
12) Backblaze agent logs (+ memory usage)
13) Run ALL
q)  Quit
```

## 🧪 Example Output
- Every section includes a `[Scanning...]` and `[Finished]` message
- Backblaze memory usage is displayed like:
  ```
  PID: 23509   RSS: 345004   KB (337.00 MB)  CPU: 1.2   CMD: /Library/Backblaze/bzserv
  ```

## ⚠️ Notes
- Run as `sudo` for full log access and completeness
- Script output is saved automatically to `/tmp/macos_diagnostics_<timestamp>.log`
- Works best on macOS 12+ (Monterey and newer)

## 📁 License
MIT License

## 🔗 Repository
[https://github.com/yourname/macspector](https://github.com/yourname/macspector)
 
 ```bash
#!/bin/bash
# macspector.sh — macOS Background Log Inspector
# https://github.com/yourname/macspector
#
# A menu-driven diagnostic tool to analyze Spotlight, memory pressure,
# indexing daemons, and system-level logs using macOS's unified logging system.

# ANSI Colors
GREEN="\033[0;32m"
YELLOW="\033[1;33m"
RESET="\033[0m"

DURATION="${1:-30m}"
LOG_FILE="/tmp/macos_diagnostics_$(date +%Y%m%d_%H%M%S).log"

print_help() {
  echo ""
  echo "macspector.sh - macOS System Diagnostic Log Reader"
  echo "----------------------------------------------------------"
  echo "This script allows you to query macOS's unified logs for key background services that impact memory,"
  echo "performance, file system indexing, and stability. It is especially useful for troubleshooting Spotlight,"
  echo "fseventsd, swap/memory pressure, LaunchDaemon failures, and third-party background tools."
  echo ""
  echo "Usage:"
  echo "  ./macspector.sh [duration]"
  echo "    duration: Optional. Log timeframe to search, e.g. 10m, 1h, 24h (default: 30m)"
  echo ""
  echo "Menu Options:"
  echo "  1  SpotlightKnowledged         - Spotlight's high-level logic and indexing controller"
  echo "  2  MDS Stores                   - The actual metadata database writer (can leak RAM)"
  echo "  3  FSEvents                     - Filesystem change tracker (triggers Spotlight, Time Machine, etc.)"
  echo "  4  BackupD                      - Time Machine-related process (core backup engine)"
  echo "  5  Swap and VM Pressure         - Logs about memory pressure and swap behavior"
  echo "  6  Launchd Failures             - Crashes or repeated daemon restarts"
  echo "  7  Metadata Worker Crashes      - mdworker/mdimporter failures"
  echo "  8  High Memory Warnings         - Jetsam and highwater alerts"
  echo "  9  WindowServer                 - Desktop environment issues and crashes"
  echo " 10  User Login & LoginWindow     - Logs related to user login sessions"
  echo " 11  APFS Snapshot Logs           - Snapshots & diskmanagementd events"
  echo " 12  Backblaze Agent Logs         - Resource spikes or abnormal behavior"
  echo " 13  Run ALL                      - Run all the above queries in sequence"
  echo "  q  Quit"
  echo ""
  echo "Log files are saved to /tmp with timestamps, e.g.:"
  echo "  $LOG_FILE"
  echo ""
  echo "GitHub: https://github.com/yourname/macspector"
  echo "License: MIT"
  echo "----------------------------------------------------------"
}

print_menu() {
  echo ""
  echo -e "✨ ${GREEN}macspector.sh${RESET} - Choose a log category to inspect"
  echo "Duration: $DURATION (change by passing as first argument, e.g. ./macspector.sh 1h)"
  echo "Logs will be saved to: $LOG_FILE"
  echo ""
  echo "h) Help"
  echo "1) Spotlight - spotlightknowledged"
  echo "2) MDS - mds_stores (indexing engine)"
  echo "3) FSEvents - fseventsd (filesystem monitor)"
  echo "4) Backup - backupd (Time Machine engine)"
  echo "5) Swap and memory pressure"
  echo "6) LaunchDaemon failures"
  echo "7) Metadata worker crashes"
  echo "8) High memory warnings"
  echo "9) WindowServer issues"
  echo "10) User Login & LoginWindow"
  echo "11) APFS Snapshot logs"
  echo "12) Backblaze agent logs"
  echo "13) Run ALL"
  echo "q) Quit"
  echo -n "> Choose an option: "
}

run_log_query() {
  local name="$1"
  local predicate="$2"
  echo -e "\n\n🔍 ===== ${YELLOW}$name${RESET} ($predicate) =====\n" | tee -a "$LOG_FILE"
  echo -e "${YELLOW}[Scanning $name logs for $DURATION...]${RESET}" | tee -a "$LOG_FILE"
  log show --predicate "$predicate" --info --debug --style syslog --last "$DURATION" 2>/dev/null | tee -a "$LOG_FILE"
  echo -e "${GREEN}[Finished $name]${RESET}\n" | tee -a "$LOG_FILE"
}

while true; do
  print_menu
  read -r choice
  case $choice in
    h|H) print_help ;;
    1)  run_log_query "SpotlightKnowledged"         'process == "spotlightknowledged"' ;;
    2)  run_log_query "MDS Stores"                  'process == "mds_stores"' ;;
    3)  run_log_query "FSEvents"                    'process == "fseventsd"' ;;
    4)  run_log_query "BackupD (Time Machine)"      'process == "backupd"' ;;
    5)  run_log_query "VM/Swap Pressure"            'eventMessage CONTAINS[c] "com.apple.vmpressure" OR eventMessage CONTAINS[c] "memory pressure"' ;;
    6)  run_log_query "Launchd Failures"            'eventMessage CONTAINS[c] "exited with code" OR eventMessage CONTAINS[c] "could not spawn"' ;;
    7)  run_log_query "Metadata Worker Crashes"     'process CONTAINS[c] "mdworker" OR process CONTAINS[c] "mdimporter"' ;;
    8)  run_log_query "High Memory Warnings"        'eventMessage CONTAINS[c] "highwater" OR eventMessage CONTAINS[c] "jetsam"' ;;
    9)  run_log_query "WindowServer"                'process == "WindowServer"' ;;
    10) run_log_query "User Login & LoginWindow"     'process == "loginwindow" || process == "login"' ;;
    11) run_log_query "APFS Snapshot Logs"          'eventMessage CONTAINS[c] "snapshot" OR process == "diskmanagementd"' ;;
    12)
        run_log_query "Backblaze Agent Logs" 'process CONTAINS[c] "bzserv" OR process CONTAINS[c] "bztransmit" OR process CONTAINS[c] "bzfilelist"'
        echo -e "${YELLOW}[Checking Backblaze memory usage with ps...]${RESET}" | tee -a "$LOG_FILE"
        ps aux | grep -E "bz(serv|transmit|filelist)" | grep -v grep | awk '{
            mem_mb = $6 / 1024;
            printf "PID: %-6s  RSS: %-8s KB (%.2f MB)  CPU: %-4s  CMD: %s\n", $2, $6, mem_mb, $3, $11
        }' | tee -a "$LOG_FILE"
        echo -e "${GREEN}[Backblaze memory usage check complete]${RESET}\n" | tee -a "$LOG_FILE"
        ;;
    13)
      run_log_query "SpotlightKnowledged"         'process == "spotlightknowledged"'
      run_log_query "MDS Stores"                  'process == "mds_stores"'
      run_log_query "FSEvents"                    'process == "fseventsd"'
      run_log_query "BackupD"                     'process == "backupd"'
      run_log_query "VM/Swap Pressure"            'eventMessage CONTAINS[c] "com.apple.vmpressure" OR eventMessage CONTAINS[c] "memory pressure"'
      run_log_query "Launchd Failures"            'eventMessage CONTAINS[c] "exited with code" OR eventMessage CONTAINS[c] "could not spawn"'
      run_log_query "Metadata Worker Crashes"     'process CONTAINS[c] "mdworker" OR process CONTAINS[c] "mdimporter"'
      run_log_query "High Memory Warnings"        'eventMessage CONTAINS[c] "highwater" OR eventMessage CONTAINS[c] "jetsam"'
      run_log_query "WindowServer"                'process == "WindowServer"'
      run_log_query "User Login & LoginWindow"     'process == "loginwindow" || process == "login"'
      run_log_query "APFS Snapshot Logs"          'eventMessage CONTAINS[c] "snapshot" OR process == "diskmanagementd"'
      run_log_query "Backblaze Agent Logs"        'process CONTAINS[c] "bzserv" OR process CONTAINS[c] "bztransmit" OR process CONTAINS[c] "bzfilelist"'
      ;;
    q|Q) echo "Exiting."; exit 0 ;;
    *) echo "Invalid option." ;;
  esac

done


```
