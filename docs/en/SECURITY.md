# Reporting security issues

If you find a security issue in Xing Janus (for example a way around Cerberus, reading a secret file, or running outside the sandbox), please do not open a public issue.

Report it privately with **Security → Report a vulnerability** in the releases repo (GitHub's private vulnerability reporting), and include:

- the version (`xj --version`) and the output of `xj doctor --report`
- how to reproduce it (ideally a small example project)
- what you expected to happen and what actually happened

Do not attach real keys or the contents of secret files. What Xing Janus blocks and what it does not: see [cerberus.md](cerberus.md).
