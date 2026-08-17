# AutoDL Host Notes

Use these notes only when the remote target is the user's `autodl` SSH alias.

- Connect through the `autodl` SSH alias.
- Treat the instance, GPU, filesystem, Python environment, and installed tools as changeable; inspect them before relying on prior observations.
- Sessions commonly start under `/root`, but verify the working directory and destination before transferring files.
- Non-login shells may have a minimal `PATH`; use a login-aware shell such as `bash -lc` when needed.
- Choose transfer, job-control, logging, and monitoring tools from what the current instance actually provides.
