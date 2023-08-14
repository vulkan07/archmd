# Server setups using Arch

## Firewall
**iptables** is a pre-installed simple firewall for Arch.
`# iptables -L` will list the rules.

> NOTE: You have to save the rules to make them persistent over reboots. Use: `# iptables-save > /etc/iptables/iptables.rules`
