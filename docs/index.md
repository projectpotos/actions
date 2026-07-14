---
title: Home
hide:
  - navigation
  - toc
---

<section class="tx-hero">
  <div class="tx-hero__content">
    <h1>♻️ Potos GitHub Actions</h1>
    <p>
      Reusable workflows and composite actions for Project Potos CI/CD pipelines.<br>
      All pipelines come from this repository: one place where third-party actions
      are selected, SHA-pinned, audited, and kept up to date.
    </p>
    <a href="getting-started/" title="Getting Started" class="md-button md-button--primary">
      Get started
      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16" width="16" height="16" fill="currentColor" style="vertical-align:middle;margin-left:.4em"><path d="M1 8a.5.5 0 0 1 .5-.5h11.793l-3.147-3.146a.5.5 0 0 1 .708-.708l4 4a.5.5 0 0 1 0 .708l-4 4a.5.5 0 0 1-.708-.708L13.293 8.5H1.5A.5.5 0 0 1 1 8z"/></svg>
    </a>
    <a href="workflows/" title="Browse Workflows" class="md-button">
      Browse Workflows
    </a>
  </div>
</section>

<section class="tx-features">
  <div class="tx-features__grid">

    <div class="tx-feature">
      <div class="tx-feature__icon">📦</div>
      <h2>Ansible Collections</h2>
      <p>
        Sanity checks, ansible-lint, molecule tests and releases for collections.
      </p>
    </div>

    <div class="tx-feature">
      <div class="tx-feature__icon">🐳</div>
      <h2>Container Images</h2>
      <p>
        Build images and push them to <strong>GitHub Packages</strong>, with a keyless
        <strong>build-provenance attestation</strong> attached to every pushed image.
      </p>
    </div>

    <div class="tx-feature">
      <div class="tx-feature__icon">🔍</div>
      <h2>Lint</h2>
      <p>
        One generic lint suite: <strong>yamllint</strong>, <strong>actionlint</strong>,
        <strong>zizmor</strong>, <strong>ruff</strong>, <strong>hadolint</strong>, and
        <strong>shellcheck</strong>.
      </p>
    </div>

    <div class="tx-feature">
      <div class="tx-feature__icon">📖</div>
      <h2>Zensical Docs</h2>
      <p>
        Build and deploy <strong>Zensical</strong> documentation sites to GitHub Pages.
      </p>
    </div>

    <div class="tx-feature">
      <div class="tx-feature__icon">🚀</div>
      <h2>Semantic Release</h2>
      <p>
        Automated releases with <strong>go-semantic-release</strong> and Conventional
        Commits.
      </p>
    </div>

    <div class="tx-feature">
      <div class="tx-feature__icon">🔒</div>
      <h2>Least-Privilege by Default</h2>
      <p>
        Every workflow declares the minimum <code>permissions</code> it needs, and the
        security docs map all controls to the <strong>ENISA Technical Advisory</strong>
        for secure package use.
      </p>
    </div>

  </div>
</section>