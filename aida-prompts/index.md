---
layout: default
title: AIDA Prompt Marketplace
---

<h1 class="my-4 text-center">List of Prompts</h1>

<!-- Search Input -->
<div class="input-group mb-3">
  <input type="text" id="searchInput" class="form-control" placeholder="Filter by tag..." onkeyup="filterPrompts()">
  <div class="input-group-append">
    <span class="input-group-text"><i class="bi bi-search"></i></span>
  </div>
</div>

<!-- Prompts Grid -->
<div class="row">
  {% assign prompts = site.prompts | sort: 'title' %}
  {% for prompt in prompts %}
    <!-- Capture the raw content of the prompt file -->
    {% capture prompt_raw_content %}
      {% include_relative {{ prompt.path }} %}
    {% endcapture %}

    <!-- Remove YAML front matter -->
    {% assign parts = prompt_raw_content | split: '---' %}
    {% if parts.size > 2 %}
      {% assign content_length = parts.size | minus: 2 %}
      {% assign prompt_content_array = parts | slice: 2, content_length %}
      {% assign prompt_content = prompt_content_array | join: '---' | strip %}
    {% else %}
      {% assign prompt_content = prompt_raw_content %}
    {% endif %}

    <div class="col-md-4 mb-4">
      <div class="prompt-box" data-toggle="modal" data-target="#promptModal{{ forloop.index }}">
        <div class="prompt-content">
          <!-- Header with title and copy button -->
          <div class="d-flex justify-content-between align-items-center mb-2">
            <h5 class="mb-1">{{ prompt.title }}</h5>
            <!-- Copy button with icon -->
            <button
              class="btn btn-outline-secondary btn-sm copy-btn"
              title="Copy Prompt"
            >
              <i class="bi bi-clipboard"></i>
            </button>
          </div>
          <!-- Prompt text -->
          <pre class="plain-text prompt-text">{{ prompt_content | xml_escape }}</pre>
          {% if prompt.tags %}
            <div class="tags">
              {% for tag in prompt.tags %}
                <a href="#" class="badge badge-primary tag-filter" data-tag="{{ tag }}">{{ tag }}</a>
              {% endfor %}
            </div>
          {% endif %}
        </div>

        <!-- Modal -->
        <div class="modal fade" id="promptModal{{ forloop.index }}" tabindex="-1" role="dialog" aria-labelledby="promptModalLabel{{ forloop.index }}" aria-hidden="true">
          <div class="modal-dialog modal-lg" role="document">
            <div class="modal-content">
              <div class="modal-header">
                <h5 class="modal-title" id="promptModalLabel{{ forloop.index }}">{{ prompt.title }}</h5>
                <button type="button" class="close" data-dismiss="modal" aria-label="Close" onclick='event.stopPropagation();'>
                  <span aria-hidden="true">&times;</span>
                </button>
              </div>
              <div class="modal-body">
                <pre class="plain-text prompt-text">{{ prompt_content | xml_escape }}</pre>
                {% if prompt.tags %}
                  <div class="tags">
                    {% for tag in prompt.tags %}
                      <a href="#" class="badge badge-primary tag-filter" data-tag="{{ tag }}">{{ tag }}</a>
                    {% endfor %}
                  </div>
                {% endif %}
              </div>
              <div class="modal-footer">
                <button
                  class="btn btn-primary copy-btn"
                >
                  <i class="bi bi-clipboard"></i> Copy Prompt
                </button>
              </div>
            </div>
          </div>
        </div>
        <!-- End Modal -->
      </div>
    </div>
  {% endfor %}
</div>

<!-- JavaScript Functions -->
<script>
  document.addEventListener('DOMContentLoaded', function() {
    // Copy to Clipboard function with Toast notification
    function copyToClipboard(text) {
      navigator.clipboard.writeText(text).then(function() {
        $('#copyToast').toast('show');
      }, function(err) {
        alert('Could not copy text: ', err);
      });
    }

    // Add event listeners to copy buttons
    var copyButtons = document.querySelectorAll('.copy-btn');
    copyButtons.forEach(function(button) {
      button.addEventListener('click', function(e) {
        e.stopPropagation();
        // Find the closest prompt container (either .prompt-box or .modal-content)
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
    function filterPrompts() {
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

    // Existing code for tag filtering
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