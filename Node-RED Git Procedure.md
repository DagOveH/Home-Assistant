  Node-RED Git Procedure

  After making changes in Node-RED

  On the HA host (Studio Code Server terminal) — one command:
  ~/sync-nodered.sh
  Or with a descriptive message:
  ~/sync-nodered.sh "What you changed"

  On Windows (optional — only needed if you want me to analyze the latest flows):
  cd ~/code/Node-RED-Home
  git pull

  ---
  That's it

  The ~/sync-nodered.sh script handles everything on the HA host: pulls latest, copies flows.json from Node-RED, commits, and
  pushes. You never need to manually git add/commit/push on the HA host again.

  Windows is read-only — it only ever does git pull, never commits or pushes.
