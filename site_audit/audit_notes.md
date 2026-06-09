# Portfolio Audit Notes

Screenshots captured:
- `01-home-desktop.png`
- `02-home-mobile.png`
- `03-projects-desktop.png`

## Priority Findings

1. The contact button uses `mailto:your.email@example.com` in `index.qmd`, so recruiters cannot contact you from the homepage.
2. The homepage says you are currently pursuing the RMIT master's degree, but the resume now says the degree was completed in June 2026.
3. The hero copy is broad and generic. It should immediately position you as a graduate AI/software engineer with RAG, computer vision, and GSoC experience.
4. The mobile homepage has horizontal overflow. The hero card and profile image are wider than a 390px viewport.
5. The skills section lists Rust, Kubernetes, HuggingFace, TensorFlow, React, and Docker. Keep only skills you can defend with current projects or add evidence for them.
6. The homepage omits the multimodal RAG project from Selected Projects even though it is one of the strongest AI signals.
7. The GSoC section has duplicate wording around reviewer feedback and system-call support.
8. Project pages need stronger outcomes, screenshots, architecture diagrams, and links to demos/source.

## Recommended Next Changes

- Replace the hero paragraph with a tighter AI recruiter pitch.
- Fix the contact email.
- Change education copy to "Completed Master of Artificial Intelligence, RMIT, June 2026."
- Put Multimodal RAG first, DashboardLLM second, Aerial AI third, CHIP-8 fourth.
- Add GitHub and live/demo links to every project card.
- Fix mobile overflow with responsive `box-sizing`, width constraints, and smaller hero padding.
- Rewrite project pages around: problem, stack, architecture, what I built, results, limitations, next steps.
