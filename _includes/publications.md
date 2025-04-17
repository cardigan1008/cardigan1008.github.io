<h2 id="publications" style="margin: 2px 0px 10px;">Publications</h2>

<div class="publications">
<ol class="bibliography">

{% for link in site.data.publications.main %}

<li style="margin-bottom: 1rem;">
  <div class="pub-row" style="padding-left: 5px;">

    {% if link.conference_short %} 
    <abbr class="badge">{{ link.conference_short }}</abbr>
    {% endif %}

    <div class="title" style="margin-top: 0.4rem;">
      {% if link.pdf %}
      <a href="{{ link.pdf }}">{{ link.title }}</a>
      {% else %}
      {{ link.title }}
      {% endif %}
    </div>

    <div class="author" style="margin-top: 0.2rem;">{{ link.authors }}</div>

    <div class="periodical" style="margin-top: 0.2rem;"><em>{{ link.conference }}</em></div>

    {% if link.pdf or link.code or link.page or link.bibtex or link.notes or link.others %}
    <div class="links" style="margin-top: 0.3rem;">
      {% if link.pdf %} 
      <a href="{{ link.pdf }}" class="btn btn-sm z-depth-0" role="button" target="_blank" style="font-size:12px;">PDF</a>
      {% endif %}
      {% if link.code %} 
      <a href="{{ link.code }}" class="btn btn-sm z-depth-0" role="button" target="_blank" style="font-size:12px;">Code</a>
      {% endif %}
      {% if link.page %} 
      <a href="{{ link.page }}" class="btn btn-sm z-depth-0" role="button" target="_blank" style="font-size:12px;">Project Page</a>
      {% endif %}
      {% if link.bibtex %} 
      <a href="{{ link.bibtex }}" class="btn btn-sm z-depth-0" role="button" target="_blank" style="font-size:12px;">BibTex</a>
      {% endif %}
      {% if link.notes %} 
      <strong><i style="color:#e74d3c">{{ link.notes }}</i></strong>
      {% endif %}
      {% if link.others %} 
      {{ link.others }}
      {% endif %}
    </div>
    {% endif %}

  </div>
</li>

{% unless forloop.last %}
<hr style="margin: 0.6rem 0; border: none; border-top: 1px solid #eee;">
{% endunless %}

{% endfor %}

</ol>
</div>
