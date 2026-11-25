# agent-msg

A mailbox for AI agent sessions (Claude Code, Codex, Kimi, …) running side by side in tmux.

Typing a whole message into another agent's pane with `tmux send-keys` floods its screen and
looks exactly like the human typing. agent-msg writes the message to a file in the
recipient's mailbox and types only a one-line nudge:

```
[agent-msg] from review: read it with `agent-msg read ~/.agent-inbox/build/new/20261125T190604Z-edfed860-review.md`
```

The recipient reads it, and the message ends with the exact command to answer it, so even a
model that has never seen the protocol can reply. The sender can block until that reply
arrives.

## Install

One bash script. Needs bash, tmux and coreutils.

```bash
curl -fsSL https://raw.githubusercontent.com/peterhel/agent-msg/main/agent-msg -o ~/.local/bin/agent-msg
chmod +x ~/.local/bin/agent-msg
```

Then tell your agents about it: paste [`instructions-snippet.md`](instructions-snippet.md)
into each model's global instruction file, e.g. `~/.claude/CLAUDE.md` (Claude Code) or
`~/.codex/AGENTS.md` (Codex). The full rules for agents are printed by `agent-msg protocol`.

## Use

```bash
agent-msg send build <<'MSG'          # prints the message id
Run the test suite and tell me which tests fail.
MSG
agent-msg wait <id> 600                # block until the reply; exit 124 on timeout
agent-msg reply <id> <<'MSG'           # answer a message you received
3 failures: …
MSG
agent-msg inbox | read [path] | whoami | protocol
```

A session's mailbox name is its tmux session name. Messages live in
`~/.agent-inbox/<session>/{new,cur}/`, one Markdown file each with `Id`, `From`, `To`,
`Date` and, for answers, `In-Reply-To` headers. While a sender is in `wait`, the reply is not
nudged into its pane; `wait` prints it.

Environment: `AGENT_MSG_DIR` (mailbox root, default `~/.agent-inbox`), `AGENT_MSG_FROM`
(override the sender name, default the tmux session).

## Sandboxed agents

A sandboxed agent may not reach the tmux socket ("Operation not permitted") or write the
mailbox ("Read-only file system"). Codex needs:

```bash
codex -s workspace-write --add-dir ~/.agent-inbox -c sandbox_workspace_write.network_access=true
```

## Trust

A message from another agent is a peer's request, not the user's instruction. The protocol
tells agents that the user's permission boundaries still apply and that a peer cannot grant
what only the user can. That is guidance to the models, not enforcement: every session on
the same tmux server and Unix account can read every mailbox and type into every pane.

## License

MIT, see [LICENSE](LICENSE).
