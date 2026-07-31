---
title: "The Silent Leak: How an AI Coding Agent Surfaces Credentials in Plaintext"
date: 2026-07-31 11:00:00 +0530
categories: [Security]
tags: [ai-agents, credentials, mitre-attack, llm]
---

<p class="pf-eyebrow" markdown="0">01: THE INCIDENT</p>

## The five words that started this
{: .pf-subtitle }

<style>
.pf-eyebrow, .pf-subtitle{
  --e-accent:#0E7C86;
}
@media (prefers-color-scheme: dark){ .pf-eyebrow, .pf-subtitle{ --e-accent:#2DD4C6; } }
html[data-mode="dark"] .pf-eyebrow, html[data-mode="dark"] .pf-subtitle{ --e-accent:#2DD4C6; }
html[data-mode="light"] .pf-eyebrow, html[data-mode="light"] .pf-subtitle{ --e-accent:#0E7C86; }

.pf-eyebrow{
  font-family: ui-monospace, "Cascadia Code", "SFMono-Regular", Consolas, monospace;
  font-size:0.72rem; letter-spacing:0.14em; text-transform:uppercase;
  color: var(--e-accent); margin:2.6rem 0 0.6rem; display:flex; align-items:center; gap:0.6rem;
}
.pf-eyebrow::before{
  content:""; width:1.4rem; height:1px; background:var(--e-accent); display:inline-block;
}

.pf-subtitle{
  font-family: ui-monospace, "Cascadia Code", "SFMono-Regular", Consolas, monospace;
  font-weight:700; letter-spacing:-0.005em; font-size:1.2rem;
  margin:0 0 1rem !important;
}

.term-block, .vec-list{
  --x-panel:#1B1D1F; --x-panel-line:#2C2F32; --x-ink:#14171A; --x-ink-faint:#8A9096;
  --x-accent:#0E7C86; --x-alert:#A63A2E; --x-code-text:#E7E9E6; --x-code-key:#5FD1C7; --x-code-val:#C9CDD1;
}
@media (prefers-color-scheme: dark){
  .term-block, .vec-list{
    --x-panel:#0B0C0D; --x-panel-line:#232628; --x-ink:#E7E9E6; --x-ink-faint:#666D72;
    --x-accent:#2DD4C6; --x-alert:#E2685A; --x-code-text:#E7E9E6; --x-code-key:#5FD1C7; --x-code-val:#C9CDD1;
  }
}
html[data-mode="dark"] .term-block, html[data-mode="dark"] .vec-list{
  --x-panel:#0B0C0D; --x-panel-line:#232628; --x-ink:#E7E9E6; --x-ink-faint:#666D72;
  --x-accent:#2DD4C6; --x-alert:#E2685A; --x-code-text:#E7E9E6; --x-code-key:#5FD1C7; --x-code-val:#C9CDD1;
}
html[data-mode="light"] .term-block, html[data-mode="light"] .vec-list{
  --x-panel:#1B1D1F; --x-panel-line:#2C2F32; --x-ink:#14171A; --x-ink-faint:#8A9096;
  --x-accent:#0E7C86; --x-alert:#A63A2E; --x-code-text:#E7E9E6; --x-code-key:#5FD1C7; --x-code-val:#C9CDD1;
}

.term-block{
  background: var(--x-panel); border:1px solid var(--x-panel-line);
  border-radius:8px; margin:1.6rem 0; overflow:hidden;
}
.term-block .term-bar{
  display:flex; align-items:center; gap:0.4rem; padding:0.55rem 0.8rem;
  border-bottom:1px solid var(--x-panel-line);
}
.term-block .term-dot{ width:8px; height:8px; border-radius:50%; background: var(--x-panel-line); }
.term-block .term-label{
  margin-left:0.5rem; font-family: ui-monospace, "Cascadia Code", "SFMono-Regular", Consolas, monospace;
  font-size:0.7rem; color: var(--x-ink-faint); letter-spacing:0.06em;
}
.term-block .term-body{ padding:1rem 1.1rem 1.15rem; overflow-x:auto; }
.term-block .term-body pre{
  margin:0; font-family: ui-monospace, "Cascadia Code", "SFMono-Regular", Consolas, monospace;
  font-size:0.86rem; line-height:1.65; color: var(--x-code-text); white-space:pre;
  background:transparent; border:none; padding:0;
}
.term-block .k{ color: var(--x-code-key); }
.term-block .v{ color: var(--x-code-val); }
.term-block .redact{
  background: var(--x-alert); color:transparent; border-radius:2px; padding:0 0.35em; user-select:none;
}
.term-block .cap{
  font-family: ui-monospace, "Cascadia Code", "SFMono-Regular", Consolas, monospace;
  font-size:0.74rem; color: var(--x-ink-faint); margin-top:0.6rem; padding:0 1.1rem 1rem;
}

.vec-list{ list-style:none; margin:1.6rem 0; padding:0; display:flex; flex-direction:column; }
.vec-list li{
  display:grid; grid-template-columns: 9.5rem 1fr; gap:1rem; padding:0.95rem 0;
  border-top:1px solid var(--x-panel-line); align-items:start;
}
.vec-list li:last-child{ border-bottom:1px solid var(--x-panel-line); }
.vec-list .tag{
  font-family: ui-monospace, "Cascadia Code", "SFMono-Regular", Consolas, monospace;
  font-size:0.72rem; letter-spacing:0.06em; color: var(--x-accent); padding-top:0.2rem;
}
.vec-list p{
  margin:0; font-size:1rem; line-height:1.65; color: var(--x-ink);
  font-family: Georgia, "Iowan Old Style", "Palatino Linotype", "Book Antiqua", serif;
}
</style>

I asked an AI agent to fix a git submodule that wasn't populating in a repos. Nothing dramatic, a five-minute ask. It found the submodule had never been initialized, worked through a slow and repeatedly stalling clone, and at one point ran a standard diagnostic to rule out authentication as the cause. The kind of thing any of us would type without thinking twice:

<div class="term-block" markdown="0">
<div class="term-bar"><span class="term-dot"></span><span class="term-dot"></span><span class="term-dot"></span><span class="term-label">sh</span></div>
<div class="term-body"><pre>$ echo "url=https://git.internal.example.com" | git credential fill
<span class="k">protocol</span>=<span class="v">https</span>
<span class="k">host</span>=<span class="v">git.internal.example.com</span>
<span class="k">username</span>=<span class="v">alice</span>
<span class="k">password</span>=<span class="redact">##########</span></pre></div>
<div class="cap">password redacted for this write-up, the real output was a valid, working credential</div>
</div>

I hadn't asked for a credential. I hadn't typed one. It was just sitting there in my chat transcript, in the clear. So I asked the only question that made sense: *how you got the creds?*

Turns out the answer wasn't dramatic at all. `git credential fill` is documented git plumbing the same call git makes internally whenever an HTTPS operation needs to authenticate. No jailbreak. No injection. No bug in sight. The agent ran it exactly the way it's meant to be run, and that's what makes it worth writing about: **nothing malicious happened, nothing unusual was typed**, just an agent with shell access doing ordinary, well-meant work, in a transcript that outlives the moment it was created.

What turned an uncomfortable moment into an actual write-up was the obvious follow-up: if this door was this ordinary, how many others look just like it? That question is what the rest of this piece is answering. Section 03 covers why having an agent as the operator changes the risk at all. Section 04 is the catalog that came out of going door to door, mapped against MITRE ATT&CK. Section 05 is the checklist for closing them.

<p class="pf-eyebrow" markdown="0">02: THE MECHANISM</p>

## At rest vs. in transit
{: .pf-subtitle }

Credential managers genuinely encrypt secrets at rest. Windows Credential Manager, macOS Keychain, `pass`, `git-credential-libsecret` on Linux (backed by the encrypted GNOME Keyring or KDE Wallet) all do this honestly. That part isn't where things go wrong.

Worth a quick footnote: `git-credential-cache` doesn't belong on that list. It never encrypts anything, it just holds the plaintext in a background daemon's memory for a configurable window (15 minutes by default), protected only by Unix-socket file permissions. Different mechanism, same ending: something, somewhere, ends up holding the plaintext.

The real problem is in the *interface*. Every credential-helper protocol has a "fill" or "get" operation, and its whole job is to decrypt and hand back plaintext, because whatever's consuming the credential needs the real value to authenticate. That handoff is supposed to happen invisibly, between two trusted local processes. It was never meant to touch a terminal a human can read, let alone a language model's context window.

An agent with shell access breaks that invisibility, and not through some flaw in how it's built, just by existing the way it does. It doesn't consume a credential internally the way git does. It runs the command as a subprocess and reads whatever comes back on stdout as plain data to reason over. So whatever the protocol hands back, the agent now holds as text, no different, as far as it's concerned, from any other line of output.

<p class="pf-eyebrow" markdown="0">03: WHY IT'S DIFFERENT NOW</p>

## What changes when the operator is an agent
{: .pf-subtitle }

Run `git credential fill` in a one-off shell script and you've got the same theoretical risk. What actually makes agentic tooling different is everything that happens to the output *after* the command returns.

<ul class="vec-list" markdown="0">
<li><span class="tag">[PERSISTENCE]</span><p>Shell output in a terminal disappears when the window closes. Agent transcripts are built to persist, as conversation history, as memory the agent writes to disk, as shared links, as logs kept for debugging.</p></li>
<li><span class="tag">[NO WATCHER]</span><p>Background and asynchronous agents run unattended for minutes or hours. A secret can surface and be written to a log well before any human reviews the session.</p></li>
<li><span class="tag">[NO JUDGMENT]</span><p>The agent has no innate sense that a line of output is sensitive. It treats <code>password=xyz</code> exactly like a directory listing, just tokens to reason about. Redaction has to come from outside the model, not from it.</p></li>
<li><span class="tag">[RETRY]</span><p>An agent chasing a flaky connection may re-run the same "helpful" diagnostic several times. Each run is a fresh exposure event, not a repeated view of the same one.</p></li>
<li><span class="tag">[PROPAGATION]</span><p>A transcript pasted into a bug report, exported for a teammate, or fed into a summarization pipeline carries the secret with it, well past the context where it first appeared.</p></li>
</ul>

<p class="pf-eyebrow" markdown="0">04: THE CATALOG</p>

## A taxonomy of surfacing vectors
{: .pf-subtitle }

`git credential fill` is one door. It's nowhere near the only one. What follows splits them into two groups: vectors with a confirmed ATT&CK sub-technique, and vectors that leak just as reliably but that the framework simply hasn't named yet. As far as I could find while researching this, some of these vectors have a clear, confirmed technique to point to, and others don't, though they still broadly fall under one.

<style>
.att-table{
  --t-panel:#1B1D1F; --t-panel-line:#2C2F32; --t-ink-soft:#565D63; --t-ink-faint:#8A9096;
  --t-accent:#0E7C86; --t-code-text:#E7E9E6; --t-code-key:#5FD1C7; --t-code-val:#C9CDD1;
}
@media (prefers-color-scheme: dark){
  .att-table{
    --t-panel:#0B0C0D; --t-panel-line:#232628; --t-ink-soft:#9BA3A8; --t-ink-faint:#666D72;
    --t-accent:#2DD4C6; --t-code-text:#E7E9E6; --t-code-key:#5FD1C7; --t-code-val:#C9CDD1;
  }
}
html[data-mode="dark"] .att-table{
  --t-panel:#0B0C0D; --t-panel-line:#232628; --t-ink-soft:#9BA3A8; --t-ink-faint:#666D72;
  --t-accent:#2DD4C6; --t-code-text:#E7E9E6; --t-code-key:#5FD1C7; --t-code-val:#C9CDD1;
}
html[data-mode="light"] .att-table{
  --t-panel:#1B1D1F; --t-panel-line:#2C2F32; --t-ink-soft:#565D63; --t-ink-faint:#8A9096;
  --t-accent:#0E7C86; --t-code-text:#E7E9E6; --t-code-key:#5FD1C7; --t-code-val:#C9CDD1;
}
.att-table h3.tblhead{
  display:flex; align-items:baseline; gap:0.6rem;
  font-family: Georgia, "Iowan Old Style", "Palatino Linotype", "Book Antiqua", serif;
  font-style: italic; font-weight:400; font-size:1.02rem;
  color: var(--t-ink-soft); margin:1.8rem 0 0.8rem;
}
.att-table h3.tblhead .tag{ font-style:normal; }
.att-table .tag{
  font-family: ui-monospace, "Cascadia Code", "SFMono-Regular", Consolas, monospace;
  font-size:0.72rem; letter-spacing:0.06em; color: var(--t-accent);
}
.att-table .tblwrap{
  overflow-x:auto; margin:0 0 1rem; border-radius:8px;
  background: var(--t-panel); border:1px solid var(--t-panel-line);
}
.att-table table{
  border-collapse:collapse; width:100%; min-width:640px; table-layout:fixed;
  font-family: ui-monospace, "Cascadia Code", "SFMono-Regular", Consolas, monospace;
  font-size:0.82rem;
}
.att-table thead th{
  text-align:left; font-size:0.7rem; letter-spacing:0.08em; text-transform:uppercase;
  color: var(--t-ink-faint); padding:0.85rem 1rem; border-bottom:1px solid var(--t-panel-line);
  white-space:nowrap;
}
.att-table thead th:nth-child(1){ width:20%; }
.att-table thead th:nth-child(2){ width:34%; }
.att-table thead th:nth-child(3){ width:26%; } 
.att-table thead th:nth-child(4){ width:20%; }
.att-table tbody tr:nth-child(2n),
.att-table tbody tr:nth-child(2n + 1){
  /* Theme's zebra-striping vars aren't symmetric between light/dark
     (--tb-even-bg is dark-only, --tb-odd-bg is near-white in light mode),
     so it paints light stripes over this table when switching themes. */
  background-color: transparent !important;
}
.att-table tbody td{
  padding:0.9rem 1rem; color: var(--t-code-val); border-bottom:1px solid var(--t-panel-line);
  vertical-align:top; line-height:1.55;
}
.att-table tbody tr:last-child td{ border-bottom:none; }
.att-table td.cat{ color: var(--t-code-text); white-space:nowrap; }
.att-table td.ex{ color: var(--t-code-key); }
.att-table td.ex code{ color:inherit; background:none; padding:0; border-radius:0; font-size:inherit; }
.att-table td.why{ color: var(--t-ink-faint); font-family: Georgia, "Iowan Old Style", serif; font-size:0.84rem; }
.att-table td.ttp{
  color: var(--t-accent); font-size:0.78rem;
  /* Beat the site-wide "wrap long table text" rule (!important), which
     otherwise breaks IDs like T1552.001 mid-word onto a second line. */
  white-space: nowrap !important;
  word-break: keep-all !important;
}
.att-table td.ttp.approx{ color: var(--t-ink-faint); }
.att-table td.ttp a{ color:inherit; text-decoration-color: currentColor; opacity:0.9; }
.att-table td.ttp a:hover{ opacity:1; }
.att-table .tblnote{
  font-size:0.92rem; color: var(--t-ink-soft); margin:0.9rem 0 1.6rem 0;
  font-family: Georgia, "Iowan Old Style", "Palatino Linotype", "Book Antiqua", serif;
}
</style>
<div class="att-table" markdown="0">

<h3 class="tblhead"><span class="tag">[TTP]</span> Confirmed ATT&CK sub-technique</h3>
<div class="tblwrap">
<table>
<thead>
<tr><th>Category</th><th>Example commands</th><th>Why it leaks</th><th>ATT&CK</th></tr>
</thead>
<tbody>
<tr><td class="cat">Credential-helper introspection</td><td class="ex"><code>git credential fill</code><br><code>security find-generic-password -w</code><br><code>Get-StoredCredential</code> (CredentialManager PS module)</td><td class="why">The command's entire purpose is decrypt-and-print.</td><td class="ttp"><a href="https://attack.mitre.org/techniques/T1555/">T1555</a>,<br><a href="https://attack.mitre.org/techniques/T1555/001/">T1555.001</a>,<br><a href="https://attack.mitre.org/techniques/T1555/004/">T1555.004</a></td></tr>
<tr><td class="cat">Password manager/browser stores</td><td class="ex">Chrome "Login Data" sqlite + OS decrypt call (DPAPI/Keychain)</td><td class="why">The stored blob is encrypted at rest, but the decryption key is scoped to the logged-in OS user, any process running as that user can decrypt it the same way the browser does.</td><td class="ttp"><a href="https://attack.mitre.org/techniques/T1555/003/">T1555.003</a></td></tr>
<tr><td class="cat">Unencrypted private key files</td><td class="ex"><code>cat ~/.ssh/id_rsa</code> (no passphrase)</td><td class="why">A passphrase-less key is usable the instant it's read, no decrypt step required.</td><td class="ttp"><a href="https://attack.mitre.org/techniques/T1552/004/">T1552.004</a></td></tr>
<tr><td class="cat">Environment/config dumps</td><td class="ex"><code>env</code><br><code>printenv</code><br><code>cat ~/.netrc</code><br><code>cat ~/.git-credentials</code><br><code>cat ~/.aws/credentials</code></td><td class="why">Secrets are stored as plaintext files or variables by design.</td><td class="ttp"><a href="https://attack.mitre.org/techniques/T1552/001/">T1552.001</a></td></tr>
<tr><td class="cat">Package manager auth files</td><td class="ex"><code>cat .npmrc</code><br><code>cat .pypirc</code><br><code>cat NuGet.Config</code></td><td class="why">Registry tokens commonly live in plaintext config so the package manager can authenticate non-interactively.</td><td class="ttp"><a href="https://attack.mitre.org/techniques/T1552/001/">T1552.001</a></td></tr>
<tr><td class="cat">Kubernetes secret stores</td><td class="ex"><code>kubectl get secret -o yaml</code></td><td class="why">Base64 is encoding, not encryption,trivially reversible.</td><td class="ttp"><a href="https://attack.mitre.org/techniques/T1552/007/">T1552.007</a></td></tr>
<tr><td class="cat">Process & history introspection</td><td class="ex"><code>ps aux</code><br>shell history files<br><code>docker inspect</code></td><td class="why">CLI arguments and history aren't treated as secret-safe by most tools.</td><td class="ttp"><a href="https://attack.mitre.org/techniques/T1552/003/">T1552.003</a>,<br><a href="https://attack.mitre.org/techniques/T1057/">T1057</a>,<br><a href="https://attack.mitre.org/techniques/T1552/007/">T1552.007</a></td></tr>
<tr><td class="cat">Cloud metadata/token endpoints</td><td class="ex"><code>curl 169.254.169.254/...</code> (token hop for IMDSv2)<br><code>gcloud auth print-access-token</code></td><td class="why">Instance-role credentials are fetched in cleartext by design.</td><td class="ttp"><a href="https://attack.mitre.org/techniques/T1552/005/">T1552.005</a></td></tr>
<tr><td class="cat">Browser/session capture</td><td class="ex">screenshots<br>HAR file exports<br>DOM dumps during automation</td><td class="why">Auth headers, cookies, and autofilled passwords are visible in the captured artifact even when never printed to a terminal.</td><td class="ttp"><a href="https://attack.mitre.org/techniques/T1113/">T1113</a>,<br><a href="https://attack.mitre.org/techniques/T1539/">T1539</a></td></tr>
</tbody>
</table>
</div>
<p class="tblnote">All nine sit under the Credential Access tactic (<strong>TA0006</strong>) except <code>T1057</code> (Process Discovery, <strong>TA0007</strong>, Discovery) and <code>T1113</code> (Screen Capture, <strong>TA0009</strong>, Collection).</p>

<h3 class="tblhead"><span class="tag">[POSSIBLE TTP]</span> No dedicated sub-technique, closest parent shown</h3>
<div class="tblwrap">
<table>
<thead>
<tr><th>Category</th><th>Example commands</th><th>Why it leaks</th><th>Closest ATT&CK</th></tr>
</thead>
<tbody>
<tr><td class="cat">Infrastructure-as-code state</td><td class="ex"><code>terraform show</code><br><code>cat terraform.tfstate</code></td><td class="why">Provisioned-resource secrets are frequently written into state files unencrypted.</td><td class="ttp approx"><a href="https://attack.mitre.org/techniques/T1552/001/">T1552.001</a></td></tr>
<tr><td class="cat">Verbose/debug flags</td><td class="ex"><code>curl -v</code><br><code>GIT_CURL_VERBOSE=1 git fetch</code></td><td class="why">Debug tracing exists specifically to print headers, including Authorization, that are normally hidden.</td><td class="ttp approx"><a href="https://attack.mitre.org/techniques/T1552/">T1552</a></td></tr>
<tr><td class="cat">Cross-process environment reads</td><td class="ex"><code>cat /proc/&lt;pid&gt;/environ</code> (same user or root)</td><td class="why">Any process's environment, including secrets passed to it, is readable by anything sharing its user permissions.</td><td class="ttp approx"><a href="https://attack.mitre.org/techniques/T1003/007/">T1003.007</a></td></tr>
<tr><td class="cat">Version-control archaeology</td><td class="ex"><code>git log -p --all</code><br><code>git show &lt;old-sha&gt;</code><br><code>git stash list</code></td><td class="why">A secret "removed" from the current branch tip often still lives in git's object database.</td><td class="ttp approx"><a href="https://attack.mitre.org/techniques/T1552/001/">T1552.001</a></td></tr>
<tr><td class="cat">CI/CD pipeline logs</td><td class="ex"><code>echo $SECRET_VAR</code><br>partial/transformed variable interpolation</td><td class="why">Log masking is pattern-based; a re-encoded or substring-echoed secret slips past the mask.</td><td class="ttp approx"><a href="https://attack.mitre.org/techniques/T1552/001/">T1552.001</a></td></tr>
<tr><td class="cat">Error messages & stack traces</td><td class="ex">Unhandled exceptions with connection strings</td><td class="why">Debug-friendly errors often embed the secret used to connect.</td><td class="ttp approx"><a href="https://attack.mitre.org/techniques/T1552/001/">T1552.001</a></td></tr>
<tr><td class="cat">URLs with embedded auth</td><td class="ex">Presigned URLs, API keys in query strings</td><td class="why">The secret <em>is</em> the URL, no special command needed at all.</td><td class="ttp approx"><a href="https://attack.mitre.org/techniques/T1552/001/">T1552.001</a></td></tr>
</tbody>
</table>
</div>

</div>

Every technique here still falls under `T1552` (Unsecured Credentials, TA0006) at the parent level. Enterprise ATT&CK is built around hosts and networks, and it simply hasn't caught up to source-code-native vectors like git history mining, CI/CD log masking, or debug-flag output, none of those have their own sub-technique yet. Read the ID here as "closest neighbor," not "exact match."

MITRE's AI-specific extension, [ATLAS](https://atlas.mitre.org), mirrors the two anchor techniques above as `AML.T0055` (Unsecured Credentials) and `AML.T0090` (OS Credential Dumping). Its newer agent and tool-invocation-specific credential-harvesting additions don't have stable published IDs yet, so I've left them out rather than guess.

A deliberate omission: SSH agent forwarding isn't in either table. It's a real risk, just a different kind, a forwarded agent lets a remote host use your key to authenticate without the key's bytes ever leaving your machine. That's someone misusing a credential, not a credential leaking in plaintext, and it earns its own write-up rather than a spot in this list.

<p class="pf-eyebrow" markdown="0">05: THE RESPONSE</p>

## What actually needs monitoring
{: .pf-subtitle }

Telling an agent to "be careful" is a policy. It's not a control. The mitigations that actually hold in practice sit between the shell and the model's context, not inside the model's judgment:

<ul class="vec-list" markdown="0">
<li><span class="tag">[GUARDRAIL]</span><p>Deny-list, or require explicit human approval for, known secret-revealing patterns: <code>credential fill/approve</code>, <code>*/.netrc</code>, <code>*/.git-credentials</code>, unfiltered <code>env</code>. Cheapest, highest-leverage control available.</p></li>
<li><span class="tag">[SCANNER]</span><p>Run every tool's stdout/stderr through a secret scanner, entropy and regex rules, the same class used by gitleaks or trufflehog, before it re-enters the model's context. Matches get replaced with a token like <code>[REDACTED:aws_secret_key]</code>, never shown.</p></li>
<li><span class="tag">[SPLIT LOG]</span><p>Keep the full raw output in an access-controlled audit log for humans. The model, and anything downstream of its context, including memory and shared transcripts, sees only the redacted version.</p></li>
<li><span class="tag">[SAFER CHECK]</span><p>Prefer verification that confirms without revealing: <code>git ls-remote</code> or <code>aws sts get-caller-identity</code> confirm a credential is valid without printing it. Reach for these before anything with "fill," "show," or "print" semantics.</p></li>
<li><span class="tag">[SHORT-LIVED]</span><p>A surfaced long-lived PAT is compromised for as long as it's valid, often months. OIDC/STS-issued tokens shrink that window to minutes-to-an-hour, capping the damage of any single exposure.</p></li>
<li><span class="tag">[SANDBOX]</span><p>Ideally the agent's shell never has direct query access to the real credential store. Secrets get injected only at the point of use, by a broker process the agent can't introspect.</p></li>
<li><span class="tag">[ROTATE]</span><p>Treat any exposure as compromised the moment it appears in a transcript, encrypted-at-rest storage of the transcript doesn't change that calculus. Rotate first; investigate after.</p></li>
<li><span class="tag">[CANARY]</span><p>Seed a small number of fake, monitored credentials in the store. A hit on one is a strong signal that something, human or agent, is enumerating secrets it shouldn't.</p></li>
</ul>

<p class="pf-eyebrow" markdown="0">CLOSING</p>

## The honest takeaway
{: .pf-subtitle }

Every credential-helper protocol has a plaintext handoff built into it, with or without AI anywhere near the loop. That's not new, and it's not really the point. The point is how ordinary the moment was: an agent doing routine, well-meant troubleshooting reached for that handoff without any signal, to itself or to anyone watching, that this step was different in kind from listing a directory.

> The fix isn't a smarter agent. It's a boundary the agent can't reason its way around: control over which commands are allowed to run, and a scanner standing between what the shell returns and what the model is permitted to remember.

Until that boundary is standard in agentic tooling, assume the conservative case: any credential that touches an agent's shell is one command away from ending up in a transcript, a log, or a shared link. Provision, scope, and rotate it like that's already true.
