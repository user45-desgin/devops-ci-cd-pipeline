               ┌───────────────────────────────┐
               │          Developer            │
               │  (pushes code to GitHub)      │
               └──────────────┬────────────────┘
                              │
                              ▼
                   ┌────────────────────────┐
                   │   GitHub Actions CI/CD  │
                   │-------------------------│
                   │ - Runs Tests (pytest)   │
                   │ - Builds Docker Image   │
                   │ - Pushes to Docker Hub  │
                   │ - SSH Deploys to EC2    │
                   └──────────────┬──────────┘
                                  │
                       SSH & Docker Pull/Run
                                  │
                                  ▼
                   ┌──────────────────────────┐
                   │     EC2 Instance         │
                   │--------------------------│
                   │ - Runs Flask Container   │
                   │ - Sends Logs to CW Logs  │
                   │ - Publishes Metrics      │
                   │   (CPU, Mem, Disk, etc.) │
                   └──────────────┬───────────┘
                                  │
                ┌─────────────────┴─────────────────┐
                │                                   │
                ▼                                   ▼
   ┌──────────────────────┐             ┌──────────────────────┐
   │  CloudWatch Metrics  │             │ CloudWatch Logs      │
   │----------------------│             │----------------------│
   │ - CPU Utilization    │             │ - Syslog             │
   │ - Disk Usage         │             │ - Docker Logs        │
   │ - Memory Usage       │             │                      │
   └──────────┬───────────┘             └──────────┬───────────┘
              │                                    │
              ▼                                    │
   ┌──────────────────────────┐                    │
   │ CloudWatch Alarm (CPU>80%)│                   │
   │ Triggers SNS Notification │                   │
   └──────────┬────────────────┘                   │
              │ SNS Trigger                        │
              ▼                                    │
       ┌────────────────────────┐                  │
       │ Lambda: Auto-Healing   │                  │
       │-------------------------│                 │
       │ - Uses Paramiko (SSH)   │                 │
       │ - Restarts Flask Docker │                 │
       │   Container on EC2      │                 │
       └──────────────┬──────────┘                 │
                      │                            │
                      ▼                            │
         ┌────────────────────────────┐            │
         │ Healthy Flask App Restored │            │
         └────────────────────────────┘            │
