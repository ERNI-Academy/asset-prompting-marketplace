---
layout: default
title: AIDA Prompt Marketplace
---

<div class="container">

  <!-- Header -->
  <h1 class="my-5 text-center">{{ page.title }}</h1>

  <!-- Search Input -->
  <div class="input-group mb-4">
    <input type="text" id="searchInput" class="form-control" placeholder="Filter by tag..." onkeyup="filterPrompts()">
    <span class="input-group-text"><i class="bi bi-search"></i></span>
  </div>

  <!-- Prompts Grid -->
  <div class="row row-cols-1 row-cols-md-2 row-cols-lg-3 g-4">
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

      <!-- Prompt Card -->
      <div class="col">
        <div class="card h-100 prompt-box" data-bs-toggle="modal" data-bs-target="#promptModal{{ forloop.index }}">
          <div class="card-body prompt-content">
            <div class="d-flex justify-content-between align-items-start mb-2">
              <h5 class="card-title">{{ prompt.title }}</h5>
              <button class="btn btn-outline-secondary btn-sm copy-btn" title="Copy Prompt">
                <i class="bi bi-clipboard"></i>
              </button>
            </div>
            <pre class="plain-text prompt-text flex-grow-1">{{ prompt_content | xml_escape }}</pre>
            {% if prompt.tags %}
              <div class="tags mt-3">
                {% for tag in prompt.tags %}
                  <a href="#" class="badge bg-primary tag-filter" data-tag="{{ tag }}">{{ tag }}</a>
                {% endfor %}
              </div>
            {% endif %}
          </div>
        </div>
      </div>

      <!-- Prompt Modal -->
      <div class="modal fade" id="promptModal{{ forloop.index }}" tabindex="-1" aria-labelledby="promptModalLabel{{ forloop.index }}" aria-hidden="true">
        <div class="modal-dialog modal-dialog-centered modal-lg">
          <div class="modal-content">
            <div class="modal-header">
              <h5 class="modal-title" id="promptModalLabel{{ forloop.index }}">{{ prompt.title }}</h5>
              <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close" onclick='event.stopPropagation();'></button>
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
        e.stopPropagation();
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
      var filters = input.split(',').map(function(item) { return item.trim(); }).filter(Boolean);
      var prompts = document.getElementsByClassName('prompt-box');

      for (var i = 0; i < prompts.length; i++) {
        var tags = prompts[i].querySelector('.tags') ? prompts[i].querySelector('.tags').innerText.toLowerCase() : '';
        var match = filters.every(function(filter) {
          return tags.includes(filter);
        });

        if (match || filters.length === 0) {
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
        e.stopPropagation();
        document.getElementById('searchInput').value = e.target.getAttribute('data-tag');
        filterPrompts();
      });
    });
  });
</script>