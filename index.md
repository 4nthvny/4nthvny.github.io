---
layout: page
title: "Anthony Rivera" \n "Cybersecurity"
permalink: /
---

<style>
/* === FULL-WIDTH GRID BREAKOUT === */
/* Break out of Alembic's narrow column and use full viewport width */
.home-grid {
  display: grid;
  grid-template-columns: minmax(0, 1.2fr) minmax(0, 1fr);
  gap: 40px;

  /* full-bleed trick: ignore parent max-width */
  width: 100vw;
  max-width: none;
  position: relative;
  left: 50%;
  right: 50%;
  margin-left: -50vw;
  margin-right: -50vw;

  margin-top: 3rem;
  margin-bottom: 3rem;
  padding: 0 5vw;  /* breathing room on sides */
}

/* Stack on mobile */
@media (max-width: 880px) {
  .home-grid {
    grid-template-columns: 1fr;
  }
}

/* LEFT: BLOG / WRITE-UPS / CERTS LIST */
.blog-list h2 {
  font-size: 1.8rem;
  color: #39ff14;
  margin-top: 10px;
  padding-top: 0;
  margin-bottom: 1rem;
}

.post-preview {
  margin-bottom: 1.5rem;
  padding-bottom: 1rem;
  border-bottom: 1px solid #222;
}

.post-preview a {
  color: #39ff14;
  text-decoration: none;
  font-size: 1.2rem;
  font-weight: 600;
}

.post-preview a:hover {
  text-decoration: underline;
}

.post-date {
  color: #888;
  font-size: 0.85rem;
  margin-top: 4px;
}

.post-excerpt {
  color: #ccc;
  font-size: 0.95rem;
  margin-top: 8px;
}

/* RIGHT: TERMINAL COLUMN */
.terminal-col {
  display: flex;
  flex-direction: column;
  align-items: stretch;
  padding-top: 0;
}

/* === TERMINAL STYLES === */
#terminal {
  background: #000;
  color: #bdbcb9;
  padding: 20px;
  border: 2px solid #39ff14; /* neon green border */
  border-radius: 0;
  font-family: "Fira Code", ui-monospace, monospace;
  line-height: 1.5;
  width: 100%;
  height: 400px;
  overflow-y: auto;
  box-sizing: border-box;
  magin-top: 0;
}

.prompt { color:#39ff14; font-weight:600; }
.cursor { animation: blink 1s steps(1) infinite; color:#39ff14; margin-left:4px; }
@keyframes blink { 50% { opacity: 0 } }
.line { margin: 0 0 .25rem 0; white-space: pre-wrap; font-size: 1.05rem; }
.input-wrap { display:flex; gap:.6rem; align-items:center; }
.input { outline:none; display:inline-block; color:#fff; min-width:4px; }

/* remove double caret */
#input-line {
  caret-color: transparent;
}
</style>

<div class="home-grid">
  <!-- LEFT SIDE: WRITE-UPS / BLOGS / CERTS -->
  <div class="blog-list">
    <h1>Recent Posts</h1>

    {%- assign sorted_pages = site.pages | sort: "date" | reverse -%}
    {%- assign shown = 0 -%}

    {%- for post in sorted_pages -%}
      {%- if post.url -%}
        {%- if post.url contains '/blog/' or post.url contains '/writeups/' or post.url contains '/certifications/' -%}
          <div class="post-preview">
            <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
            <div class="post-date">
              {{ post.date | date: "%B %d, %Y" }}
            </div>
            <p class="post-excerpt">
              {{
                post.excerpt
                  | default: post.content
                  | markdownify
                  | strip_html
                  | replace: "\n", " "
                  | replace: "*", ""
                  | replace: "#", ""
                  | replace: "`", ""
                  | replace: ">", ""
                  | strip
                  | truncate: 160
              }}
            </p>
          </div>

          {%- assign shown = shown | plus: 1 -%}
          {%- if shown >= 2 -%}
            {%- break -%}
          {%- endif -%}
        {%- endif -%}
      {%- endif -%}
    {%- endfor -%}
  </div>

  <!-- RIGHT SIDE: TERMINAL + BUTTONS -->
  <div class="terminal-col">

    <!-- START: Web Terminal -->
    <div id="terminal" aria-label="web terminal"></div>

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
        const parts = raw.split(/\s+/);
        const cmd = (parts.shift() || '').toLowerCase();

        const simple = {
          'help': () => `Available commands:
- whoami 
- ls 
- cat flag.txt
- base64 
- certs`,
          'whoami': () => 'Hi, Im Anthony :D',
          'ls': () => 'flag.txt',
          'certs': () => `Certifications:
- CompTIA Security+
- Red Team Operator
- Blue Team Level 1`,
          'clear': () => { term.innerHTML = ''; return ''; }
        };

        // base64 command
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

  </div>
</div>
