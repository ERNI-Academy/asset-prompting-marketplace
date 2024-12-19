---
layout: default
---

# List of Prompts

<!-- Search Input -->
<input type="text" id="searchInput" placeholder="Filter by tag..." onkeyup="filterPrompts()">

<!-- Prompts List -->
<ul id="promptList">
  {% assign prompts = site.prompts | sort: 'title' %}
  {% for prompt in prompts %}
    <li data-tags="{{ prompt.tags | join: ' ' }}">
      <h2>{{ prompt.title }}</h2>
      {% assign author_name = prompt.author.name | default: "Unknown" %}
      {% if prompt.author.url %}
        <p class="author"><strong>Author:</strong> <a href="{{ prompt.author.url }}">{{ author_name }}</a></p>
      {% else %}
        <p class="author"><strong>Author:</strong> {{ author_name }}</p>
      {% endif %}
      {% if prompt.date %}
        <p><strong>Date:</strong> {{ prompt.date | date: "%B %d, %Y" }}</p>
      {% endif %}
      <pre>{{ prompt.content }}</pre>
      <button onclick="copyToClipboard(`{{ prompt.content | strip_newlines | escape }}`)">Copy</button>
    </li>
  {% endfor %}
</ul>

<!-- JavaScript Functions -->
<script>
  function copyToClipboard(text) {
    navigator.clipboard.writeText(text).then(function() {
      alert('Prompt copied to clipboard');
    }, function(err) {
      alert('Could not copy text: ', err);
    });
  }

  function filterPrompts() {
  var input = document.getElementById('searchInput').value.toLowerCase();
  var filters = input.split(',').map(function(item) { return item.trim(); });
  var ul = document.getElementById('promptList');
  var li = ul.getElementsByTagName('li');

  for (var i = 0; i < li.length; i++) {
    var tags = li[i].getAttribute('data-tags').toLowerCase();
    var match = filters.every(function(filter) {
      return tags.indexOf(filter) > -1;
    });

    if (match || input === '') {
      li[i].style.display = "";
    } else {
      li[i].style.display = "none";
    }
  }
}
</script>