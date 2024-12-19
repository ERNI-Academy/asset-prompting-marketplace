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
    <div class="col-md-4 mb-4">
      <div class="prompt-box" data-toggle="modal" data-target="#promptModal{{ forloop.index }}">
        <div class="prompt-content">
          <h5 class="mb-1">{{ prompt.title }}</h5>
          <!-- Copy button with icon -->
          <button class="btn btn-outline-secondary btn-sm float-right" onclick='copyToClipboard({{ prompt.content | jsonify }}); event.stopPropagation();' title="Copy Prompt">
            <i class="bi bi-clipboard"></i>
          </button>
          <pre class="plain-text">{{ prompt.content }}</pre>
          {% if prompt.tags %}
            <div class="tags">
              {% for tag in prompt.tags %}
                <a href="#" class="badge badge-primary tag-filter" data-tag="{{ tag }}" onclick="event.stopPropagation();">{{ tag }}</a>
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
                <pre class="plain-text">{{ prompt.content }}</pre>
                {% if prompt.tags %}
                  <div class="tags">
                    {% for tag in prompt.tags %}
                      <a href="#" class="badge badge-primary tag-filter" data-tag="{{ tag }}">{{ tag }}</a>
                    {% endfor %}
                  </div>
                {% endif %}
              </div>
              <div class="modal-footer">
                <button class="btn btn-primary" onclick='copyToClipboard({{ prompt.content | jsonify }});'>
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
  // Copy to Clipboard function with Toast notification
  function copyToClipboard(text) {
    navigator.clipboard.writeText(text).then(function() {
      $('#copyToast').toast('show');
    }, function(err) {
      alert('Could not copy text: ', err);
    });
  }

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

  // Add event listeners to tags for filtering
  document.addEventListener('DOMContentLoaded', function() {
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