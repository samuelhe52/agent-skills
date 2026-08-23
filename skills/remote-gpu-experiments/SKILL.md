---
name: remote-gpu-experiments
description: Operate ML and research workloads on remote GPU hosts. Use when an agent needs to inspect a remote GPU environment, transfer code or data, launch and monitor jobs, troubleshoot runtime compatibility, or retrieve experiment results over SSH or a workload scheduler.
---

# Remote GPU Experiments

Adapt the execution strategy to the actual host, repository, and user request.

## Requirements

- Inspect the relevant local code and remote environment before making changes.
- Detect the host's runtime, GPU tooling, storage layout, and job-control mechanism instead of assuming them.
- Preserve existing local and remote data. Verify synchronization targets before using options that remove or overwrite files.
- MANDATORY: Use a **durable execution mechanism** that makes the job outlive the SSH session. for long-running jobs and retain useful logs.
- After launching work, provide the commands needed to inspect, attach to, and stop it.
- Avoid installing packages or rebuilding the remote environment unless necessary and within the requested scope.
- When comparing variants, keep conditions comparable and report the results as a comparison.

## Tool Selection

- Use `ssh` for remote inspection and command execution.
- Use `rsync` for repeated directory transfers; use `scp`, `sftp`, or an equivalent tool for simple one-off transfers.
- Use an available persistent session or process manager such as `tmux`. If the host is a managed cluster, beware of the cluster's scheduler and job submission system. Follow system/project-specific guidelines.
- Use diagnostics appropriate to the host, such as scheduler status commands, `nvidia-smi`, or `rocm-smi`.
- Capture stdout and stderr in durable logs, and use an appropriate transfer tool to retrieve results.

## Host References

Read [references/autodl-preset.md](references/autodl-preset.md) only when working with the user's `autodl` SSH target.
