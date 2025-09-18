---
layout: page
title: ""
permalink: /
---
<div class="center-page">
  <h1>Anthony Rivera</h1>
  <p><strong>I Like Computers :D</strong><p>
  <p><strong>Idk what else to write here tbh…</strong></p>

  <div id="terminal" aria-label="web terminal"></div>

<!-- START: Web Terminal -->
<style>
/* Center everything in the middle of the viewport */
.center-page {
  display: flex;
  flex-direction: column;
  align-items: center;      /* horizontal centering */
  justify-content: center;  /* vertical centering */
  min-height: 100vh;        /* fill full screen height */
  text-align: center;
  gap: 12px;                /* space between elements */
}

#terminal {
  background: #000;
  color: #bdbcb9;
  padding: 20px;
  border: 2px solid #39ff14; /* neon green border */
  border-radius: 0; 
  font-family: "Fira Code", ui-monospace, monospace;
  line-height: 1.5;
  width: 100%;        /* takes full width of container */
  max-width: 1100px;  /* optional: cap width */
  height: 400px;      /* <- fixed height */
  overflow-y: auto;   /* <- scroll when too much output */
  margin: 0rem 0;
  box-sizing: border-box;
}
  .prompt { color:#39ff14; font-weight:600; }
  .cursor { animation: blink 1s steps(1) infinite; color:#39ff14; margin-left:4px; }
  @keyframes blink { 50% { opacity: 0 } }
  .line { margin: 0 0 .25rem 0; white-space: pre-wrap; font-size: 1.05rem; }
  .input-wrap { display:flex; gap:.6rem; align-items:center; }
  .input { outline:none; display:inline-block; color:#fff; min-width:4px; }

  /* new rule to remove double cursor */
  #input-line {
    caret-color: transparent;
  }
</style>

<script>
document.addEventListener('DOMContentLoaded', () => {
  const term = document.getElementById('terminal');
  const PROMPT = 'anthony@home:~$';
  const INTRO  = 'Uhh.. Wut-amd64 x86_64 GNU/Linux\nType "help" for commands';

  // demo file contents
  const FILES = {
    'flag.txt': 'Y2hhdCB3aWxsIGkgZ2V0IGhpcmVkPw==' // base64 of "chat will i get hired?"
  };

  const HISTORY = [];
  let histIdx = -1;

  // utils
  const sanitize = (s) => { const d = document.createElement('div'); d.innerText = s; return d.innerHTML; };

  function writeLine(text, asCmd=false) {
    const p = document.createElement('p');
    p.className = 'line';
    p.innerHTML = asCmd
      ? `<span class="prompt">${sanitize(PROMPT)}</span> ${sanitize(text)}`
      : sanitize(text);
    term.appendChild(p);
    term.scrollTop = term.scrollHeight;
  }

  function placeCaretAtEnd(el){
    const r = document.createRange();
    const s = window.getSelection();
    r.selectNodeContents(el);
    r.collapse(false);
    s.removeAllRanges();
    s.addRange(r);
  }

  function writeInputLine() {
    const wrap = document.createElement('div');
    wrap.className = 'input-wrap';

    const promptSpan = document.createElement('span');
    promptSpan.className = 'prompt';
    promptSpan.textContent = PROMPT;

    const input = document.createElement('span');
    input.className = 'input';
    input.id = 'input-line';
    input.contentEditable = 'true';
    input.spellcheck = false;

    const cursor = document.createElement('span');
    cursor.className = 'cursor';
    cursor.textContent = '█';

    wrap.appendChild(promptSpan);
    wrap.appendChild(input);
    wrap.appendChild(cursor);
    term.appendChild(wrap);
    term.scrollTop = term.scrollHeight;

    setTimeout(() => { input.focus(); placeCaretAtEnd(input); }, 0);

    input.addEventListener('keydown', (e) => {
      // Ctrl+L -> clear
      if (e.ctrlKey && e.key.toLowerCase() === 'l') {
        e.preventDefault();
        term.innerHTML = '';
        writeLine(INTRO);
        writeInputLine();
        return;
      }

      // history
      if (e.key === 'ArrowUp' || e.key === 'ArrowDown') {
        e.preventDefault();
        if (!HISTORY.length) return;
        if (e.key === 'ArrowUp') histIdx = Math.max(0, histIdx - 1);
        else histIdx = Math.min(HISTORY.length - 1, histIdx + 1);
        input.textContent = HISTORY[histIdx] || '';
        placeCaretAtEnd(input);
        return;
      }

      // Enter -> execute
      if (e.key === 'Enter') {
        e.preventDefault();
        const raw = input.innerText.trim();
        term.removeChild(wrap);
        writeLine(raw, true);
        if (raw) { HISTORY.push(raw); histIdx = HISTORY.length; }
        handleCommand(raw);
        return;
      }
    });
  }

  function handleCommand(raw) {
    // tokenize, preserving flags like "-d"
    const parts = raw.split(/\s+/);
    const cmd = (parts.shift() || '').toLowerCase();

    // map of simple commands
    const simple = {
      'help': () => `Available commands:
- whoami 
- ls 
- cat flag.txt
- base64 
- certs 
- job`,
      'whoami': () => 'Hi, Im Anthony :D',
      'ls': () => 'flag.txt',
      'certs': () => `Certifications:
- CompTIA Security+
- Red Team Operator
- Blue Team Level 1`,
      'job': () => 'Uhh chat.. Pls hire me? Internships?',
      'clear': () => { term.innerHTML = ''; return ''; }
    };

    // base64 command (no arguments, just decodes the flag)
if (cmd === 'base64') {
  writeLine('chat will i get hired?');
  writeInputLine();
  return;
}
    // cat flag.txt
    if (cmd === 'cat') {
      const target = (parts.shift() || '').toLowerCase();
      if (!target) {
        writeLine('usage: cat <file>');
      } else if (target === 'flag.txt') {
        writeLine(FILES['flag.txt']);
      } else {
        writeLine(`cat: ${target}: No such file`);
      }
      writeInputLine();
      return;
    }

    // simple commands
    if (cmd in simple) {
      const out = simple[cmd]();
      if (out) writeLine(out);
      writeInputLine();
      return;
    }

    if (cmd.length) writeLine(`command not found: ${cmd}`);
    writeInputLine();
  }

  // click to focus
  term.addEventListener('click', () => {
    const i = document.getElementById('input-line');
    if (i) i.focus();
  });

  // boot
  writeLine(INTRO);
  writeInputLine();
});
</script>
<!-- END: Web Terminal -->


<div class="typeset" style="text-align:center; margin-top:2rem;">
  {% include button.html text="GitHub" icon="github" link="https://github.com/4nthvny" color="#0366d6" %}
  {% include button.html text="LinkedIn" icon="linkedin" link="https://www.linkedin.com/in/anthony-d-rivera" color="#0077B5" %}
</div>
