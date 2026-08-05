```{=html}
<div class="project-bento list">
<% for (const item of items) { %>
  <article class="project-item" <%= metadataAttrs(item) %>>
    <a class="project-link" href="<%- item.path %>">
      <div class="project-media">
        <img class="project-image" src="<%- item.image %>" alt="<%- item['image-alt'] || item.title %>" loading="lazy">
      </div>
      <div class="project-copy">
        <div class="project-meta">
          <span class="project-index"><%- String(item.order).padStart(2, '0') %></span>
          <span class="listing-project-area"><%- item['project-area'] %></span>
        </div>
        <h3 class="listing-title"><%- item.title %></h3>
        <p class="listing-description"><%- item.description %></p>
        <div class="project-tech"><%- item.tech %></div>
      </div>
    </a>
  </article>
<% } %>
</div>
```
