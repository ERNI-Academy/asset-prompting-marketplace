---
layout: default
title: AIDA Prompt Marketplace
---

<div class="container main-content-container">
  <!-- Search Input with new styling -->
  <div class="input-group mb-4">
    <input type="text" 
           id="searchInput" 
           class="form-control rounded-full" 
           placeholder="Search prompts..." 
           onkeyup="filterPrompts()">
    <span class="input-group-text rounded-full">
      <i class="bi bi-search"></i>
    </span>
  </div>

  <!-- Prompts Grid with new styling -->
  <div class="row row-cols-1 row-cols-sm-2 row-cols-md-3 g-4">
    {% assign prompts = site.prompts | sort: 'title' %}
    {% for prompt in prompts %}
      {% capture prompt_raw_content %}
        {% include_relative {{ prompt.path }} %}
      {% endcapture %}

      <!-- Process prompt content -->
      {% assign parts = prompt_raw_content | split: '---' %}
      {% if parts.size > 2 %}
        {% assign prompt_content = parts | slice: 2 | join: '---' | strip %}
      {% else %}
        {% assign prompt_content = prompt_raw_content %}
      {% endif %}

      <!-- Updated Prompt Card -->
      <div class="col">
        <div class="card h-100 prompt-box">
          <div class="card-body prompt-content d-flex flex-column">
            <div class="d-flex justify-content-between align-items-start mb-2">
              <h5 class="card-title">{{ prompt.title }}</h5>
              <button class="btn btn-outline-light btn-sm copy-btn" title="Copy Prompt">
                <i class="bi bi-clipboard"></i>
              </button>
            </div>
            <div class="modal-trigger" data-bs-toggle="modal" data-bs-target="#promptModal{{ forloop.index }}">
              <pre class="plain-text prompt-text">{{ prompt_content | strip | truncate: 200 }}</pre>
              {% if prompt.tags %}
                <div class="tags mt-3">
                  {% for tag in prompt.tags %}
                    <a href="#" class="badge bg-primary tag-filter" data-tag="{{ tag }}" onclick="event.stopPropagation();">{{ tag }}</a>
                  {% endfor %}
                </div>
              {% endif %}
            </div>
          </div>
        </div>
      </div>

      <!-- Prompt Modal -->
      <div class="modal fade" id="promptModal{{ forloop.index }}" tabindex="-1">
        <div class="modal-dialog modal-dialog-centered modal-dialog-scrollable modal-lg">
          <div class="modal-content">
            <div class="modal-header">
              <h5 class="modal-title">{{ prompt.title }}</h5>
              <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
            </div>
            <div class="modal-body">
              <pre class="plain-text prompt-text">{{ prompt_content | xml_escape }}</pre>
              {% if prompt.tags %}
                <div class="tags mt-3">
                  {% for tag in prompt.tags %}
                    <a href="#" class="badge bg-primary tag-filter" data-tag="{{ tag }}">{{ tag }}</a>
                  {% endfor %}
                </div>
              {% endif %}
            </div>
            <div class="modal-footer">
              <button class="btn btn-primary copy-btn">
                <i class="bi bi-clipboard"></i> Copy Prompt
              </button>
            </div>
          </div>
        </div>
      </div>
    {% endfor %}
  </div>
</div>

<!-- JavaScript Functions -->
<script>
  document.addEventListener('DOMContentLoaded', function() {

    // Copy to Clipboard function with Toast notification
    function copyToClipboard(text) {
      navigator.clipboard.writeText(text).then(function() {
        // Show Bootstrap 5 toast
        var toastEl = document.getElementById('copyToast');
        var toast = new bootstrap.Toast(toastEl);
        toast.show();
      }, function(err) {
        alert('Could not copy text: ', err);
      });
    }

    // Add event listeners to copy buttons
    var copyButtons = document.querySelectorAll('.copy-btn');
    copyButtons.forEach(function(button) {
      button.addEventListener('click', function(e) {
        e.stopPropagation(); // Prevent the event from bubbling up
        e.preventDefault();  // Prevent default action (if any)

        var promptContainer = this.closest('.prompt-box, .modal-content');
        if (promptContainer) {
          var promptContentElement = promptContainer.querySelector('.prompt-text');
          if (promptContentElement) {
            var text = promptContentElement.textContent || promptContentElement.innerText;
            copyToClipboard(text.trim());
          } else {
            alert('Prompt content not found.');
          }
        } else {
          alert('Prompt container not found.');
        }
      });
    });

    // Filter Prompts based on input
    window.filterPrompts = function() {
      var input = document.getElementById('searchInput').value.toLowerCase().trim();
      var prompts = document.getElementsByClassName('prompt-box');

      for (var i = 0; i < prompts.length; i++) {
        var title = prompts[i].querySelector('.card-title').innerText.toLowerCase();
        var content = prompts[i].querySelector('.prompt-text').innerText.toLowerCase();
        var tags = prompts[i].querySelector('.tags') ? prompts[i].querySelector('.tags').innerText.toLowerCase() : '';

        var match = title.includes(input) || content.includes(input) || tags.includes(input);

        if (match || input === '') {
          prompts[i].parentElement.style.display = "";
        } else {
          prompts[i].parentElement.style.display = "none";
        }
      }
    }

    // Tag filtering
    var tagLinks = document.getElementsByClassName('tag-filter');
    Array.prototype.forEach.call(tagLinks, function(link) {
      link.addEventListener('click', function(e) {
        e.preventDefault();
        e.stopPropagation(); // Prevent the modal from opening when clicking on a tag
        document.getElementById('searchInput').value = e.target.getAttribute('data-tag');
        filterPrompts();
      });
    });

  });
</script>