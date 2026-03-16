<script>
  import { onMount } from 'svelte';

  // ── CONFIG — update these for each client ──
  const CLINIC_NAME    = "SmileCare Dental Clinic";
  const DOCTOR_NAME    = "Dr. Anjali Sharma";
  const DOCTOR_QUAL    = "BDS, MDS – Prosthodontics";
  const WHATSAPP_NUM   = "919999999999"; // replace with real number
  const CITY           = "New Delhi";
  const ADDRESS        = "B-12, Sector 18, Noida – 201301";
  const EMAIL          = "info@smilecare.in";
  const CLOUDINARY_CLOUD  = "dmldsl2q4";
  const CLOUDINARY_PRESET = "dental_gallery";
  const ADMIN_PASSWORD    = "smile2024"; // change this

  // ── State ──
  let galleryImages = [];
  let adminUnlocked = false;
  let adminPassword = '';
  let adminError = '';
  let uploading = false;
  let uploadProgress = 0;
  let activeSection = 'home';
  let mobileMenuOpen = false;
  let showAdminPanel = false;

  // Load saved images from localStorage
  onMount(() => {
    const saved = localStorage.getItem('dental_gallery');
    if (saved) galleryImages = JSON.parse(saved);
    
    // Intersection observer for nav highlight
    const sections = document.querySelectorAll('section[id]');
    const observer = new IntersectionObserver((entries) => {
      entries.forEach(e => { if (e.isIntersecting) activeSection = e.target.id; });
    }, { threshold: 0.4 });
    sections.forEach(s => observer.observe(s));
  });

  function saveGallery() {
    localStorage.setItem('dental_gallery', JSON.stringify(galleryImages));
  }

  function unlockAdmin() {
    if (adminPassword === ADMIN_PASSWORD) {
      adminUnlocked = true;
      adminError = '';
    } else {
      adminError = 'Incorrect password. Please try again.';
    }
  }

  async function handleUpload(e) {
    const files = Array.from(e.target.files);
    if (!files.length) return;
    uploading = true;
    uploadProgress = 0;

    for (let i = 0; i < files.length; i++) {
      const file = files[i];
      const formData = new FormData();
      formData.append('file', file);
      formData.append('upload_preset', CLOUDINARY_PRESET);
      formData.append('folder', 'dental_gallery');

      const res = await fetch(`https://api.cloudinary.com/v1_1/${CLOUDINARY_CLOUD}/image/upload`, {
        method: 'POST',
        body: formData
      });
      const data = await res.json();
      if (data.secure_url) {
        galleryImages = [...galleryImages, {
          id: data.public_id,
          url: data.secure_url,
          label: '',
          uploadedAt: new Date().toLocaleDateString('en-IN')
        }];
        saveGallery();
      }
      uploadProgress = Math.round(((i + 1) / files.length) * 100);
    }
    uploading = false;
    e.target.value = '';
  }

  function deleteImage(id) {
    if (!confirm('Remove this image from gallery?')) return;
    galleryImages = galleryImages.filter(img => img.id !== id);
    saveGallery();
  }

  function updateLabel(id, label) {
    galleryImages = galleryImages.map(img => img.id === id ? { ...img, label } : img);
    saveGallery();
  }

  function waLink(msg = '') {
    return `https://wa.me/${WHATSAPP_NUM}?text=${encodeURIComponent(msg || `Hi ${DOCTOR_NAME}, I'd like to book an appointment at ${CLINIC_NAME}.`)}`;
  }

  const services = [
    { icon: '🦷', title: 'General Dentistry', desc: 'Routine checkups, cleanings, fillings and preventive care for the whole family.' },
    { icon: '✨', title: 'Teeth Whitening', desc: 'Professional-grade whitening treatments that deliver visible results in one session.' },
    { icon: '🔲', title: 'Dental Implants', desc: 'Permanent tooth replacement that looks, feels and functions like your natural teeth.' },
    { icon: '😁', title: 'Orthodontics', desc: 'Braces and clear aligners to straighten teeth and correct bite issues at any age.' },
    { icon: '👑', title: 'Crowns & Veneers', desc: 'Restore damaged teeth or transform your smile with custom porcelain restorations.' },
    { icon: '🩺', title: 'Root Canal', desc: 'Painless, modern root canal therapy to save infected teeth and relieve pain quickly.' },
  ];

  const testimonials = [
    { name: 'Priya Mehta', stars: 5, text: 'Absolutely transformed my smile! The team was so gentle and professional. Best dental experience I\'ve ever had.' },
    { name: 'Rahul Gupta', stars: 5, text: 'Had a root canal done here — completely painless! Dr. Sharma explained every step. Highly recommend.' },
    { name: 'Sneha Kapoor', stars: 5, text: 'The teeth whitening results were incredible. I got compliments from everyone at my wedding. Thank you!' },
  ];
</script>

<svelte:head>
  <title>{CLINIC_NAME} | {CITY}</title>
  <meta name="description" content="{DOCTOR_NAME} – {CLINIC_NAME} in {CITY}. Expert dental care, teeth whitening, implants and more. Book via WhatsApp.">
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link href="https://fonts.googleapis.com/css2?family=Cormorant+Garamond:ital,wght@0,400;0,600;0,700;1,400&family=DM+Sans:wght@300;400;500;600&display=swap" rel="stylesheet">
</svelte:head>

<!-- ═══ NAV ═══ -->
<nav class="nav" class:scrolled={true}>
  <a href="#home" class="nav-logo">
    <span class="logo-cross">✚</span>
    <span class="logo-text">{CLINIC_NAME}</span>
  </a>
  <ul class="nav-links">
    {#each [['home','Home'],['services','Services'],['gallery','Gallery'],['about','About'],['contact','Contact']] as [id, label]}
      <li><a href="#{id}" class:active={activeSection===id}>{label}</a></li>
    {/each}
  </ul>
  <a href={waLink()} target="_blank" class="nav-cta">📱 Book Appointment</a>
  <button class="hamburger" on:click={() => mobileMenuOpen = !mobileMenuOpen}>
    <span></span><span></span><span></span>
  </button>
</nav>

{#if mobileMenuOpen}
  <div class="mobile-menu">
    {#each [['home','Home'],['services','Services'],['gallery','Gallery'],['about','About'],['contact','Contact']] as [id, label]}
      <a href="#{id}" on:click={() => mobileMenuOpen=false}>{label}</a>
    {/each}
    <a href={waLink()} target="_blank" class="mobile-cta">📱 Book on WhatsApp</a>
  </div>
{/if}

<!-- ═══ HERO ═══ -->
<section id="home" class="hero">
  <div class="hero-bg">
    <div class="hero-circle c1"></div>
    <div class="hero-circle c2"></div>
    <div class="hero-dots"></div>
  </div>
  <div class="hero-content">
    <div class="hero-badge">✦ Trusted Dental Care in {CITY}</div>
    <h1>Your Perfect <em>Smile</em><br>Starts Here</h1>
    <p class="hero-sub">Expert dental treatments delivered with precision, comfort, and care. From routine cleanings to complete smile makeovers.</p>
    <div class="hero-actions">
      <a href={waLink()} target="_blank" class="btn-primary">
        <svg viewBox="0 0 24 24" width="18" height="18" fill="currentColor"><path d="M17.472 14.382c-.297-.149-1.758-.867-2.03-.967-.273-.099-.471-.148-.67.15-.197.297-.767.966-.94 1.164-.173.199-.347.223-.644.075-.297-.15-1.255-.463-2.39-1.475-.883-.788-1.48-1.761-1.653-2.059-.173-.297-.018-.458.13-.606.134-.133.298-.347.446-.52.149-.174.198-.298.298-.497.099-.198.05-.371-.025-.52-.075-.149-.669-1.612-.916-2.207-.242-.579-.487-.5-.669-.51-.173-.008-.371-.01-.57-.01-.198 0-.52.074-.792.372-.272.297-1.04 1.016-1.04 2.479 0 1.462 1.065 2.875 1.213 3.074.149.198 2.096 3.2 5.077 4.487.709.306 1.262.489 1.694.625.712.227 1.36.195 1.871.118.571-.085 1.758-.719 2.006-1.413.248-.694.248-1.289.173-1.413-.074-.124-.272-.198-.57-.347m-5.421 7.403h-.004a9.87 9.87 0 01-5.031-1.378l-.361-.214-3.741.982.998-3.648-.235-.374a9.86 9.86 0 01-1.51-5.26c.001-5.45 4.436-9.884 9.888-9.884 2.64 0 5.122 1.03 6.988 2.898a9.825 9.825 0 012.893 6.994c-.003 5.45-4.437 9.884-9.885 9.884m8.413-18.297A11.815 11.815 0 0012.05 0C5.495 0 .16 5.335.157 11.892c0 2.096.547 4.142 1.588 5.945L.057 24l6.305-1.654a11.882 11.882 0 005.683 1.448h.005c6.554 0 11.89-5.335 11.893-11.893a11.821 11.821 0 00-3.48-8.413z"/></svg>
        Book Appointment
      </a>
      <a href="#services" class="btn-ghost">Our Services →</a>
    </div>
    <div class="hero-stats">
      <div class="stat"><span class="stat-num">2000+</span><span class="stat-label">Happy Patients</span></div>
      <div class="stat-div"></div>
      <div class="stat"><span class="stat-num">15+</span><span class="stat-label">Years Experience</span></div>
      <div class="stat-div"></div>
      <div class="stat"><span class="stat-num">4.9★</span><span class="stat-label">Google Rating</span></div>
    </div>
  </div>
  <div class="hero-visual">
    <div class="hero-card">
      <div class="hcard-top">
        <div class="hcard-avatar">😁</div>
        <div>
          <div class="hcard-name">{DOCTOR_NAME}</div>
          <div class="hcard-qual">{DOCTOR_QUAL}</div>
        </div>
      </div>
      <div class="hcard-divider"></div>
      <div class="hcard-row"><span class="hcard-dot green"></span> Clinic Open Today</div>
      <div class="hcard-row"><span class="hcard-dot blue"></span> Mon – Sat · 10am – 7pm</div>
      <div class="hcard-row"><span class="hcard-dot teal"></span> Emergency: Same Day Slots</div>
      <a href={waLink()} target="_blank" class="hcard-btn">
        Chat on WhatsApp →
      </a>
    </div>
  </div>
</section>

<!-- ═══ SERVICES ═══ -->
<section id="services" class="section services-section">
  <div class="section-header">
    <span class="section-tag">What We Offer</span>
    <h2>Comprehensive <em>Dental Care</em></h2>
    <p>From preventive care to advanced cosmetic procedures — we offer everything your smile needs under one roof.</p>
  </div>
  <div class="services-grid">
    {#each services as s}
      <div class="service-card">
        <div class="svc-icon">{s.icon}</div>
        <h3>{s.title}</h3>
        <p>{s.desc}</p>
        <a href={waLink(`Hi, I'd like to know more about ${s.title}.`)} target="_blank" class="svc-link">Book This →</a>
      </div>
    {/each}
  </div>
</section>

<!-- ═══ BEFORE & AFTER GALLERY ═══ -->
<section id="gallery" class="section gallery-section">
  <div class="section-header">
    <span class="section-tag">Smile Transformations</span>
    <h2>Before &amp; <em>After</em></h2>
    <p>Real results from real patients. See the difference expert dental care can make.</p>
  </div>

  {#if galleryImages.length > 0}
    <div class="gallery-grid">
      {#each galleryImages as img}
        <div class="gallery-item">
          <img src={img.url} alt={img.label || 'Smile transformation'} loading="lazy" />
          {#if img.label}
            <div class="gallery-label">{img.label}</div>
          {/if}
        </div>
      {/each}
    </div>
  {:else}
    <div class="gallery-empty">
      <div class="empty-icon">📸</div>
      <p>Smile transformation photos coming soon!</p>
      <p style="font-size:13px;opacity:0.5">The doctor will upload before & after photos here.</p>
    </div>
  {/if}

  <div class="gallery-cta">
    <a href={waLink('Hi, I\'d like to see more about smile makeovers at your clinic.')} target="_blank" class="btn-primary">Want a Transformation? Chat With Us →</a>
  </div>
</section>

<!-- ═══ ABOUT ═══ -->
<section id="about" class="section about-section">
  <div class="about-visual">
    <div class="about-card">
      <div class="about-avatar">🦷</div>
      <div class="about-badge">✦ {DOCTOR_QUAL}</div>
      <div class="about-highlights">
        <div class="ah-item"><span class="ah-num">15+</span><span class="ah-label">Years Practice</span></div>
        <div class="ah-item"><span class="ah-num">2000+</span><span class="ah-label">Smiles Made</span></div>
        <div class="ah-item"><span class="ah-num">100%</span><span class="ah-label">Safe & Sterile</span></div>
      </div>
    </div>
  </div>
  <div class="about-content">
    <span class="section-tag">About the Doctor</span>
    <h2>Meet <em>{DOCTOR_NAME}</em></h2>
    <p>With over 15 years of experience in modern dentistry, {DOCTOR_NAME} has helped thousands of patients achieve healthy, confident smiles. Trained from one of India's premier dental institutions, she combines clinical excellence with a gentle, patient-first approach.</p>
    <p>At {CLINIC_NAME}, we believe dental care should be comfortable, transparent, and accessible. We use the latest equipment and techniques to ensure every visit is as pain-free as possible.</p>
    <div class="about-features">
      {#each ['Painless treatments with latest technology', 'Transparent pricing — no hidden costs', 'Child-friendly & family dental care', 'Emergency appointments available', 'Sterilised, hygienic environment'] as f}
        <div class="af-item"><span class="af-check">✓</span> {f}</div>
      {/each}
    </div>
    <a href={waLink()} target="_blank" class="btn-primary" style="margin-top:28px;display:inline-flex">Book a Consultation →</a>
  </div>
</section>

<!-- ═══ TESTIMONIALS ═══ -->
<div class="testi-section">
  <div class="section-header" style="margin-bottom:48px">
    <span class="section-tag">Patient Stories</span>
    <h2>What Our <em>Patients Say</em></h2>
  </div>
  <div class="testi-grid">
    {#each testimonials as t}
      <div class="testi-card">
        <div class="testi-stars">{'★'.repeat(t.stars)}</div>
        <p>"{t.text}"</p>
        <div class="testi-name">— {t.name}</div>
      </div>
    {/each}
  </div>
</div>

<!-- ═══ CONTACT ═══ -->
<section id="contact" class="section contact-section">
  <div class="section-header">
    <span class="section-tag">Get In Touch</span>
    <h2>Book Your <em>Appointment</em></h2>
    <p>The easiest way to book is via WhatsApp. We typically respond within minutes.</p>
  </div>
  <div class="contact-grid">
    <div class="contact-card wa-card">
      <div class="contact-icon">💬</div>
      <h3>WhatsApp</h3>
      <p>Chat with us directly. Share your concern, photos, or just say hello — we'll guide you from there.</p>
      <a href={waLink()} target="_blank" class="btn-primary wa-btn">
        Open WhatsApp Chat →
      </a>
    </div>
    <div class="contact-info">
      <div class="ci-item">
        <span class="ci-icon">📍</span>
        <div><strong>Address</strong><br>{ADDRESS}</div>
      </div>
      <div class="ci-item">
        <span class="ci-icon">🕐</span>
        <div><strong>Hours</strong><br>Mon – Sat: 10:00 AM – 7:00 PM<br>Sunday: Closed</div>
      </div>
      <div class="ci-item">
        <span class="ci-icon">📧</span>
        <div><strong>Email</strong><br><a href="mailto:{EMAIL}">{EMAIL}</a></div>
      </div>
    </div>
  </div>
</section>

<!-- ═══ ADMIN SECTION ═══ -->
<div class="admin-toggle-wrap">
  <button class="admin-toggle-btn" on:click={() => showAdminPanel = !showAdminPanel}>
    {showAdminPanel ? '▲ Hide' : '⚙ Admin Panel'}
  </button>
</div>

{#if showAdminPanel}
  <div class="admin-section">
    <div class="admin-box">
      <h3>🔒 Doctor Admin Panel</h3>
      <p class="admin-sub">Upload before & after photos to the gallery</p>

      {#if !adminUnlocked}
        <div class="admin-login">
          <input
            class="admin-input"
            type="password"
            bind:value={adminPassword}
            placeholder="Enter admin password"
            on:keydown={e => e.key === 'Enter' && unlockAdmin()}
          />
          <button class="btn-primary" on:click={unlockAdmin}>Unlock</button>
          {#if adminError}<div class="admin-error">{adminError}</div>{/if}
        </div>
      {:else}
        <div class="admin-panel">
          <div class="upload-area">
            <input
              type="file"
              id="file-upload"
              multiple
              accept="image/*"
              on:change={handleUpload}
              style="display:none"
            />
            <label for="file-upload" class="upload-label" class:uploading>
              {#if uploading}
                <div class="upload-spinner"></div>
                <span>Uploading... {uploadProgress}%</span>
              {:else}
                <span class="upload-icon">📤</span>
                <span>Click to upload photos</span>
                <span class="upload-hint">Supports JPG, PNG, WEBP — multiple files allowed</span>
              {/if}
            </label>
          </div>

          {#if galleryImages.length > 0}
            <div class="admin-gallery">
              <h4>Uploaded Photos ({galleryImages.length})</h4>
              <div class="admin-grid">
                {#each galleryImages as img}
                  <div class="admin-img-card">
                    <img src={img.url} alt="Gallery" />
                    <input
                      class="admin-label-input"
                      type="text"
                      placeholder="Add label (e.g. Teeth Whitening)"
                      value={img.label}
                      on:input={e => updateLabel(img.id, e.target.value)}
                    />
                    <button class="delete-btn" on:click={() => deleteImage(img.id)}>🗑 Remove</button>
                  </div>
                {/each}
              </div>
            </div>
          {:else}
            <div class="admin-empty">No photos uploaded yet. Upload your first before & after photo above.</div>
          {/if}
        </div>
      {/if}
    </div>
  </div>
{/if}

<!-- ═══ FOOTER ═══ -->
<footer>
  <div class="footer-inner">
    <div class="footer-brand">
      <span class="logo-cross">✚</span>
      <span style="font-family:'Cormorant Garamond',serif;font-size:1.2rem;font-weight:600">{CLINIC_NAME}</span>
    </div>
    <p>Caring for smiles in {CITY} since 2009. Your comfort and confidence is our priority.</p>
    <a href={waLink()} target="_blank" class="footer-wa">💬 Book on WhatsApp</a>
  </div>
  <div class="footer-bottom">
    <span>© 2026 {CLINIC_NAME}. All rights reserved.</span>
    <span>Built by <a href="https://srdncomputers.com" target="_blank">SRDN Computers</a></span>
  </div>
</footer>

<!-- WhatsApp Floating Button -->
<a href={waLink()} target="_blank" class="wa-float" title="Chat on WhatsApp">
  <svg viewBox="0 0 24 24" fill="white" width="28" height="28"><path d="M17.472 14.382c-.297-.149-1.758-.867-2.03-.967-.273-.099-.471-.148-.67.15-.197.297-.767.966-.94 1.164-.173.199-.347.223-.644.075-.297-.15-1.255-.463-2.39-1.475-.883-.788-1.48-1.761-1.653-2.059-.173-.297-.018-.458.13-.606.134-.133.298-.347.446-.52.149-.174.198-.298.298-.497.099-.198.05-.371-.025-.52-.075-.149-.669-1.612-.916-2.207-.242-.579-.487-.5-.669-.51-.173-.008-.371-.01-.57-.01-.198 0-.52.074-.792.372-.272.297-1.04 1.016-1.04 2.479 0 1.462 1.065 2.875 1.213 3.074.149.198 2.096 3.2 5.077 4.487.709.306 1.262.489 1.694.625.712.227 1.36.195 1.871.118.571-.085 1.758-.719 2.006-1.413.248-.694.248-1.289.173-1.413-.074-.124-.272-.198-.57-.347m-5.421 7.403h-.004a9.87 9.87 0 01-5.031-1.378l-.361-.214-3.741.982.998-3.648-.235-.374a9.86 9.86 0 01-1.51-5.26c.001-5.45 4.436-9.884 9.888-9.884 2.64 0 5.122 1.03 6.988 2.898a9.825 9.825 0 012.893 6.994c-.003 5.45-4.437 9.884-9.885 9.884m8.413-18.297A11.815 11.815 0 0012.05 0C5.495 0 .16 5.335.157 11.892c0 2.096.547 4.142 1.588 5.945L.057 24l6.305-1.654a11.882 11.882 0 005.683 1.448h.005c6.554 0 11.89-5.335 11.893-11.893a11.821 11.821 0 00-3.48-8.413z"/></svg>
</a>

<style>
  :global(*) { margin: 0; padding: 0; box-sizing: border-box; }
  :global(html) { scroll-behavior: smooth; }
  :global(body) {
    font-family: 'DM Sans', sans-serif;
    background: #fdfcf9;
    color: #1a1a1a;
    overflow-x: hidden;
  }
  :global(a) { text-decoration: none; color: inherit; }

  /* ── Variables ── */
  :global(:root) {
    --teal: #0d9488;
    --teal-light: #f0fdf9;
    --teal-mid: #ccfbf1;
    --gold: #b8860b;
    --cream: #fdfcf9;
    --dark: #0f1a17;
    --text: #1a1a1a;
    --muted: #6b7280;
    --border: #e5e7eb;
    --card: #ffffff;
    --serif: 'Cormorant Garamond', Georgia, serif;
  }

  /* ── Nav ── */
  .nav {
    position: fixed; top: 0; left: 0; right: 0; z-index: 100;
    display: flex; align-items: center; gap: 32px;
    padding: 18px 5%;
    background: rgba(253,252,249,0.92);
    backdrop-filter: blur(12px);
    border-bottom: 1px solid var(--border);
  }
  .nav-logo { display: flex; align-items: center; gap: 10px; font-family: var(--serif); font-size: 1.15rem; font-weight: 600; }
  .logo-cross { color: var(--teal); font-size: 1.3rem; }
  .nav-links { display: flex; gap: 28px; list-style: none; margin-left: auto; }
  .nav-links a { font-size: 0.88rem; font-weight: 500; color: var(--muted); transition: color 0.2s; }
  .nav-links a:hover, .nav-links a.active { color: var(--teal); }
  .nav-cta {
    display: inline-flex; align-items: center; gap: 6px;
    background: var(--teal); color: white;
    padding: 9px 18px; border-radius: 8px;
    font-size: 0.84rem; font-weight: 600;
    transition: all 0.2s; white-space: nowrap;
  }
  .nav-cta:hover { background: #0b7a6e; transform: translateY(-1px); }
  .hamburger { display: none; flex-direction: column; gap: 5px; background: none; border: none; cursor: pointer; padding: 4px; }
  .hamburger span { display: block; width: 22px; height: 2px; background: var(--text); border-radius: 2px; }

  .mobile-menu {
    position: fixed; top: 65px; left: 0; right: 0; z-index: 99;
    background: var(--cream); border-bottom: 1px solid var(--border);
    display: flex; flex-direction: column; padding: 16px 5%;
  }
  .mobile-menu a { padding: 12px 0; font-size: 1rem; border-bottom: 1px solid var(--border); color: var(--text); }
  .mobile-cta { color: var(--teal) !important; font-weight: 600; border-bottom: none !important; }

  /* ── Buttons ── */
  .btn-primary {
    display: inline-flex; align-items: center; gap: 8px;
    background: var(--teal); color: white;
    padding: 13px 26px; border-radius: 10px;
    font-size: 0.92rem; font-weight: 600;
    transition: all 0.2s; cursor: pointer; border: none;
  }
  .btn-primary:hover { background: #0b7a6e; transform: translateY(-2px); box-shadow: 0 8px 24px rgba(13,148,136,0.25); }
  .btn-ghost {
    display: inline-flex; align-items: center; gap: 8px;
    background: transparent; color: var(--text);
    border: 1.5px solid var(--border);
    padding: 13px 26px; border-radius: 10px;
    font-size: 0.92rem; font-weight: 500;
    transition: all 0.2s;
  }
  .btn-ghost:hover { border-color: var(--teal); color: var(--teal); }

  /* ── Hero ── */
  .hero {
    min-height: 100vh; display: grid;
    grid-template-columns: 1fr 420px;
    gap: 60px; align-items: center;
    padding: 120px 5% 80px;
    position: relative; overflow: hidden;
    background: linear-gradient(135deg, #f0fdf9 0%, #fdfcf9 60%);
  }
  .hero-bg { position: absolute; inset: 0; pointer-events: none; }
  .hero-circle {
    position: absolute; border-radius: 50%;
    background: radial-gradient(circle, rgba(13,148,136,0.08) 0%, transparent 70%);
  }
  .c1 { width: 700px; height: 700px; top: -200px; right: -100px; }
  .c2 { width: 400px; height: 400px; bottom: 0; left: -100px; }
  .hero-dots {
    position: absolute; inset: 0;
    background-image: radial-gradient(rgba(13,148,136,0.08) 1px, transparent 1px);
    background-size: 32px 32px;
    mask-image: radial-gradient(ellipse 80% 80% at 70% 50%, transparent 30%, black 100%);
  }
  .hero-content { position: relative; z-index: 1; }
  .hero-badge {
    display: inline-block;
    background: var(--teal-mid); color: var(--teal);
    padding: 6px 16px; border-radius: 100px;
    font-size: 0.78rem; font-weight: 600; letter-spacing: 0.05em;
    margin-bottom: 24px;
  }
  .hero h1 {
    font-family: var(--serif);
    font-size: clamp(2.8rem, 5vw, 4.2rem);
    font-weight: 700; line-height: 1.1;
    margin-bottom: 20px; color: var(--dark);
  }
  .hero h1 em { color: var(--teal); font-style: italic; }
  .hero-sub { font-size: 1.05rem; color: var(--muted); line-height: 1.75; max-width: 500px; margin-bottom: 36px; }
  .hero-actions { display: flex; gap: 12px; flex-wrap: wrap; margin-bottom: 48px; }
  .hero-stats { display: flex; align-items: center; gap: 24px; }
  .stat { display: flex; flex-direction: column; }
  .stat-num { font-family: var(--serif); font-size: 1.6rem; font-weight: 700; color: var(--dark); }
  .stat-label { font-size: 0.75rem; color: var(--muted); }
  .stat-div { width: 1px; height: 36px; background: var(--border); }

  /* Hero card */
  .hero-visual { position: relative; z-index: 1; }
  .hero-card {
    background: white; border: 1px solid var(--border);
    border-radius: 20px; padding: 28px;
    box-shadow: 0 20px 60px rgba(0,0,0,0.08);
  }
  .hcard-top { display: flex; gap: 14px; align-items: center; margin-bottom: 18px; }
  .hcard-avatar { font-size: 2.5rem; }
  .hcard-name { font-family: var(--serif); font-size: 1.1rem; font-weight: 600; }
  .hcard-qual { font-size: 0.78rem; color: var(--muted); margin-top: 2px; }
  .hcard-divider { height: 1px; background: var(--border); margin-bottom: 16px; }
  .hcard-row { display: flex; align-items: center; gap: 10px; font-size: 0.86rem; margin-bottom: 10px; color: var(--muted); }
  .hcard-dot { width: 8px; height: 8px; border-radius: 50%; flex-shrink: 0; }
  .hcard-dot.green { background: #10b981; box-shadow: 0 0 0 3px rgba(16,185,129,0.15); }
  .hcard-dot.blue { background: #3b82f6; }
  .hcard-dot.teal { background: var(--teal); }
  .hcard-btn {
    display: block; text-align: center; margin-top: 18px;
    background: var(--teal-light); color: var(--teal);
    border: 1.5px solid var(--teal-mid);
    padding: 11px; border-radius: 10px;
    font-size: 0.88rem; font-weight: 600;
    transition: all 0.2s;
  }
  .hcard-btn:hover { background: var(--teal); color: white; }

  /* ── Sections ── */
  .section { padding: 100px 5%; max-width: 1200px; margin: 0 auto; }
  .section-header { text-align: center; margin-bottom: 64px; }
  .section-tag {
    display: inline-block;
    color: var(--teal); font-size: 0.78rem; font-weight: 600;
    letter-spacing: 0.12em; text-transform: uppercase;
    margin-bottom: 12px;
  }
  .section-header h2 {
    font-family: var(--serif);
    font-size: clamp(2rem, 3.5vw, 2.8rem);
    font-weight: 700; color: var(--dark);
    margin-bottom: 14px; line-height: 1.2;
  }
  .section-header h2 em { color: var(--teal); font-style: italic; }
  .section-header p { color: var(--muted); font-size: 1rem; max-width: 560px; margin: 0 auto; line-height: 1.7; }

  /* ── Services ── */
  .services-section { background: #f8fffe; border-radius: 0; max-width: 100%; padding: 100px 5%; }
  .services-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 20px; max-width: 1200px; margin: 0 auto; }
  .service-card {
    background: white; border: 1px solid var(--border);
    border-radius: 16px; padding: 28px;
    transition: all 0.25s;
  }
  .service-card:hover { border-color: var(--teal); transform: translateY(-4px); box-shadow: 0 12px 40px rgba(13,148,136,0.1); }
  .svc-icon { font-size: 2rem; margin-bottom: 14px; }
  .service-card h3 { font-family: var(--serif); font-size: 1.15rem; font-weight: 600; margin-bottom: 8px; }
  .service-card p { font-size: 0.875rem; color: var(--muted); line-height: 1.7; margin-bottom: 16px; }
  .svc-link { font-size: 0.84rem; font-weight: 600; color: var(--teal); }
  .svc-link:hover { text-decoration: underline; }

  /* ── Gallery ── */
  .gallery-section { max-width: 1200px; }
  .gallery-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); gap: 16px; margin-bottom: 48px; }
  .gallery-item { position: relative; border-radius: 14px; overflow: hidden; aspect-ratio: 4/3; }
  .gallery-item img { width: 100%; height: 100%; object-fit: cover; transition: transform 0.4s; }
  .gallery-item:hover img { transform: scale(1.04); }
  .gallery-label {
    position: absolute; bottom: 0; left: 0; right: 0;
    background: linear-gradient(transparent, rgba(0,0,0,0.65));
    color: white; padding: 20px 14px 12px;
    font-size: 0.84rem; font-weight: 500;
  }
  .gallery-empty {
    text-align: center; padding: 80px 20px;
    background: var(--teal-light); border-radius: 16px;
    border: 2px dashed var(--teal-mid); margin-bottom: 48px;
  }
  .empty-icon { font-size: 3rem; margin-bottom: 12px; }
  .gallery-empty p { color: var(--muted); }
  .gallery-cta { text-align: center; }

  /* ── About ── */
  .about-section { display: grid; grid-template-columns: 400px 1fr; gap: 80px; align-items: center; max-width: 1200px; }
  .about-card {
    background: linear-gradient(135deg, var(--teal) 0%, #0b7a6e 100%);
    border-radius: 24px; padding: 40px; color: white; text-align: center;
  }
  .about-avatar { font-size: 4rem; margin-bottom: 16px; }
  .about-badge { font-size: 0.84rem; opacity: 0.85; margin-bottom: 28px; font-style: italic; }
  .about-highlights { display: grid; grid-template-columns: repeat(3, 1fr); gap: 12px; }
  .ah-item { background: rgba(255,255,255,0.12); border-radius: 12px; padding: 16px 8px; }
  .ah-num { display: block; font-family: var(--serif); font-size: 1.6rem; font-weight: 700; }
  .ah-label { display: block; font-size: 0.68rem; opacity: 0.8; margin-top: 3px; }
  .about-content h2 { font-family: var(--serif); font-size: 2.4rem; font-weight: 700; color: var(--dark); margin: 12px 0 20px; line-height: 1.2; }
  .about-content h2 em { color: var(--teal); font-style: italic; }
  .about-content p { color: var(--muted); line-height: 1.8; margin-bottom: 14px; font-size: 0.95rem; }
  .about-features { margin-top: 20px; display: flex; flex-direction: column; gap: 10px; }
  .af-item { display: flex; align-items: center; gap: 10px; font-size: 0.9rem; }
  .af-check { color: var(--teal); font-weight: 700; }

  /* ── Testimonials ── */
  .testi-section { background: var(--dark); padding: 100px 5%; }
  .testi-section .section-header h2 { color: white; }
  .testi-section .section-tag { color: #5eead4; }
  .testi-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 20px; max-width: 1200px; margin: 0 auto; }
  .testi-card {
    background: rgba(255,255,255,0.05);
    border: 1px solid rgba(255,255,255,0.08);
    border-radius: 16px; padding: 28px;
  }
  .testi-stars { color: #f5b800; font-size: 0.9rem; letter-spacing: 3px; margin-bottom: 16px; }
  .testi-card p { color: #d1d5db; font-size: 0.92rem; line-height: 1.8; font-style: italic; margin-bottom: 16px; }
  .testi-name { font-size: 0.84rem; color: #5eead4; font-weight: 600; }

  /* ── Contact ── */
  .contact-section { max-width: 900px; }
  .contact-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 24px; }
  .contact-card {
    background: white; border: 1px solid var(--border);
    border-radius: 16px; padding: 32px;
    text-align: center;
  }
  .wa-card { border-color: #25D366; background: linear-gradient(135deg, #f0fff4, white); }
  .contact-icon { font-size: 2.5rem; margin-bottom: 14px; }
  .contact-card h3 { font-family: var(--serif); font-size: 1.3rem; margin-bottom: 10px; }
  .contact-card p { color: var(--muted); font-size: 0.88rem; line-height: 1.7; margin-bottom: 20px; }
  .wa-btn { background: #25D366 !important; }
  .wa-btn:hover { background: #1ebe5a !important; }
  .contact-info { background: white; border: 1px solid var(--border); border-radius: 16px; padding: 32px; display: flex; flex-direction: column; gap: 20px; }
  .ci-item { display: flex; gap: 14px; align-items: flex-start; font-size: 0.9rem; line-height: 1.6; }
  .ci-icon { font-size: 1.2rem; flex-shrink: 0; margin-top: 2px; }
  .ci-item strong { display: block; margin-bottom: 3px; }
  .ci-item a { color: var(--teal); }

  /* ── Admin ── */
  .admin-toggle-wrap { text-align: center; padding: 24px; background: #f9fafb; border-top: 1px solid var(--border); }
  .admin-toggle-btn {
    background: none; border: 1px solid var(--border);
    color: var(--muted); padding: 8px 20px; border-radius: 8px;
    font-size: 0.82rem; cursor: pointer; transition: all 0.2s;
  }
  .admin-toggle-btn:hover { border-color: var(--teal); color: var(--teal); }

  .admin-section { background: #f0fdf9; border-top: 1px solid var(--teal-mid); padding: 60px 5%; }
  .admin-box { max-width: 800px; margin: 0 auto; }
  .admin-box h3 { font-family: var(--serif); font-size: 1.6rem; margin-bottom: 6px; }
  .admin-sub { color: var(--muted); font-size: 0.9rem; margin-bottom: 28px; }
  .admin-login { display: flex; gap: 12px; align-items: center; }
  .admin-input { flex: 1; padding: 12px 16px; border: 1.5px solid var(--border); border-radius: 10px; font-size: 0.92rem; font-family: 'DM Sans', sans-serif; outline: none; }
  .admin-input:focus { border-color: var(--teal); }
  .admin-error { color: #ef4444; font-size: 0.84rem; margin-top: 10px; }

  .upload-area { margin-bottom: 28px; }
  .upload-label {
    display: flex; flex-direction: column; align-items: center; justify-content: center;
    gap: 8px; padding: 40px;
    border: 2px dashed var(--teal-mid);
    border-radius: 14px; background: white;
    cursor: pointer; transition: all 0.2s; text-align: center;
  }
  .upload-label:hover, .upload-label.uploading { border-color: var(--teal); background: var(--teal-light); }
  .upload-icon { font-size: 2rem; }
  .upload-label span { font-size: 0.92rem; color: var(--muted); }
  .upload-hint { font-size: 0.78rem !important; opacity: 0.6; }
  .upload-spinner {
    width: 28px; height: 28px; border-radius: 50%;
    border: 3px solid var(--teal-mid);
    border-top-color: var(--teal);
    animation: spin 0.8s linear infinite;
  }
  @keyframes spin { to { transform: rotate(360deg); } }

  .admin-gallery h4 { font-family: var(--serif); font-size: 1.1rem; margin-bottom: 16px; }
  .admin-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); gap: 14px; }
  .admin-img-card { background: white; border: 1px solid var(--border); border-radius: 12px; overflow: hidden; }
  .admin-img-card img { width: 100%; aspect-ratio: 4/3; object-fit: cover; }
  .admin-label-input { width: 100%; padding: 8px 10px; border: none; border-top: 1px solid var(--border); font-size: 0.8rem; font-family: 'DM Sans', sans-serif; outline: none; }
  .delete-btn { width: 100%; padding: 8px; background: none; border: none; border-top: 1px solid var(--border); color: #ef4444; font-size: 0.8rem; cursor: pointer; transition: background 0.2s; }
  .delete-btn:hover { background: #fef2f2; }
  .admin-empty { text-align: center; color: var(--muted); font-size: 0.9rem; padding: 40px; background: white; border-radius: 12px; border: 1px solid var(--border); }

  /* ── Footer ── */
  footer { background: var(--dark); color: #d1d5db; padding: 48px 5% 0; }
  .footer-inner { max-width: 1200px; margin: 0 auto; text-align: center; padding-bottom: 36px; border-bottom: 1px solid rgba(255,255,255,0.08); }
  .footer-brand { display: flex; align-items: center; justify-content: center; gap: 10px; margin-bottom: 12px; }
  .footer-brand .logo-cross { color: #5eead4; font-size: 1.4rem; }
  footer p { font-size: 0.9rem; opacity: 0.6; margin-bottom: 20px; }
  .footer-wa { display: inline-block; background: rgba(255,255,255,0.08); color: #5eead4; padding: 10px 24px; border-radius: 8px; font-size: 0.88rem; font-weight: 600; }
  .footer-bottom { max-width: 1200px; margin: 0 auto; display: flex; justify-content: space-between; padding: 16px 0; font-size: 0.78rem; opacity: 0.4; }
  .footer-bottom a { color: inherit; text-decoration: underline; }

  /* ── WhatsApp Float ── */
  .wa-float {
    position: fixed; bottom: 28px; right: 28px; z-index: 999;
    width: 58px; height: 58px; border-radius: 50%;
    background: #25D366;
    display: flex; align-items: center; justify-content: center;
    box-shadow: 0 4px 20px rgba(37,211,102,0.4);
    transition: all 0.2s;
  }
  .wa-float:hover { transform: scale(1.1); box-shadow: 0 8px 30px rgba(37,211,102,0.5); }

  /* ── Responsive ── */
  @media (max-width: 1024px) {
    .hero { grid-template-columns: 1fr; padding-top: 100px; }
    .hero-visual { display: none; }
    .about-section { grid-template-columns: 1fr; }
    .about-visual { display: none; }
    .services-grid { grid-template-columns: repeat(2, 1fr); }
    .testi-grid { grid-template-columns: 1fr; }
  }
  @media (max-width: 640px) {
    .nav-links, .nav-cta { display: none; }
    .hamburger { display: flex; }
    .services-grid { grid-template-columns: 1fr; }
    .contact-grid { grid-template-columns: 1fr; }
    .hero h1 { font-size: 2.4rem; }
    .hero-stats { flex-wrap: wrap; gap: 16px; }
    .footer-bottom { flex-direction: column; gap: 6px; align-items: center; }
  }
</style>
