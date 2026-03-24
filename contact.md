---
layout: page
title: Contact
subtitle: "$ ping assmaa"
permalink: /contact/
---

<div class="contact-grid">
  <div>
    <div class="section-label" style="margin-bottom:1.5rem;">Get In Touch</div>
    <p>Whether you want to collaborate on a security project, discuss CTF challenges, or just say hello, my inbox is open.</p>
    <p>Response time: usually within 24–48 hours.</p>

    <div style="margin-top:2rem;">

      <div class="contact-item">
        <div class="ci-icon">@</div>
        <div class="ci-body">
          <div class="ci-label">Email</div>
          <div class="ci-value">
            <a href="mailto:assmaaZEGHAIDER58@gmail.com">assmaaZEGHAIDER58@gmail.com</a>
          </div>
        </div>
      </div>

      <div class="contact-item">
        <div class="ci-icon">GH</div>
        <div class="ci-body">
          <div class="ci-label">GitHub</div>
          <div class="ci-value">
            <a href="https://github.com/Assmaa-sec" target="_blank" rel="noopener">github.com/Assmaa-sec</a>
          </div>
        </div>
      </div>

      <div class="contact-item">
        <div class="ci-icon">LI</div>
        <div class="ci-body">
          <div class="ci-label">LinkedIn</div>
          <div class="ci-value">
            <a href="https://www.linkedin.com/in/assmaa-zeghaider-17777a303/" target="_blank" rel="noopener">linkedin.com/in/assmaa-zeghaider</a>
          </div>
        </div>
      </div>

    </div>
  </div>

  <div>
    <div class="section-label" style="margin-bottom:1.5rem;">Send a Message</div>
    <form action="https://formspree.io/f/mkoqdkno" method="POST" style="display:flex; flex-direction:column; gap:1rem;">

      <div>
        <label style="display:block; font-size:0.72rem; color:#555; letter-spacing:1.5px; text-transform:uppercase; margin-bottom:0.4rem;">Name</label>
        <input type="text" name="name" required
          style="width:100%; background:#111; border:1px solid rgba(0,255,65,0.2); color:#c8c8c8;
                 font-family:inherit; font-size:0.85rem; padding:0.6rem 0.8rem; border-radius:4px;
                 outline:none; transition:border-color 0.25s;"
          onfocus="this.style.borderColor='rgba(0,255,65,0.5)'"
          onblur="this.style.borderColor='rgba(0,255,65,0.2)'"
          placeholder="Your name">
      </div>

      <div>
        <label style="display:block; font-size:0.72rem; color:#555; letter-spacing:1.5px; text-transform:uppercase; margin-bottom:0.4rem;">Email</label>
        <input type="email" name="email" required
          style="width:100%; background:#111; border:1px solid rgba(0,255,65,0.2); color:#c8c8c8;
                 font-family:inherit; font-size:0.85rem; padding:0.6rem 0.8rem; border-radius:4px;
                 outline:none; transition:border-color 0.25s;"
          onfocus="this.style.borderColor='rgba(0,255,65,0.5)'"
          onblur="this.style.borderColor='rgba(0,255,65,0.2)'"
          placeholder="your@email.com">
      </div>

      <div>
        <label style="display:block; font-size:0.72rem; color:#555; letter-spacing:1.5px; text-transform:uppercase; margin-bottom:0.4rem;">Message</label>
        <textarea name="message" required rows="5"
          style="width:100%; background:#111; border:1px solid rgba(0,255,65,0.2); color:#c8c8c8;
                 font-family:inherit; font-size:0.85rem; padding:0.6rem 0.8rem; border-radius:4px;
                 outline:none; resize:vertical; transition:border-color 0.25s;"
          onfocus="this.style.borderColor='rgba(0,255,65,0.5)'"
          onblur="this.style.borderColor='rgba(0,255,65,0.2)'"
          placeholder="Hello..."></textarea>
      </div>

      <div>
        <button type="submit" class="btn btn-primary">Send Message</button>
      </div>

    </form>
  </div>
</div>
