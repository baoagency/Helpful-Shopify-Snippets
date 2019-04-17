## What we're creating
![https://i.imgur.com/zC1NOA5.png](https://i.imgur.com/zC1NOA5.png)

## Description
By definition on Shopify collections are all on the same level and there is no such parent / child feature. This snippet allows us to create a faux relationship like that.

## Setup

### Navigation
![https://i.imgur.com/VEL5Nss.png](https://i.imgur.com/VEL5Nss.png)

### Code
```liquid
{%- assign should_have_breadcrumb = false -%}
{%- assign breadcrumbs_linklist = linklists.collection-tree -%}
{%- assign breadcrumbs = '' -%}

{%- assign collection_handle = collection.handle -%}
{%- assign breadcrumb_link = nil -%}
{%- assign child_breadcrumb_link = nil -%}
{%- assign grandchild_breadcrumb_link = nil -%}

{%- for link in breadcrumbs_linklist.links -%}
  {%- if child_breadcrumb_link != nil -%}
    {% break %}
  {%- endif -%}

  {%- if link.type != 'collection_link' -%}
    {% continue %}
  {%- endif -%}

  {%- assign link_collection_handle = link.object.handle -%}

  {%- if link_collection_handle == collection_handle -%}
    {%- assign should_have_breadcrumb = true -%}
    {%- assign breadcrumb_link = link -%}

    {%- break -%}
  {%- endif -%}

  {%- if link.links.size > 0 -%}
    {%- for child_link in link.links -%}
      {%- if grandchild_breadcrumb_link != nil -%}
        {% break %}
      {%- endif -%}

      {%- if link.type != 'collection_link' -%}
        {% continue %}
      {%- endif -%}

      {%- assign link_collection_handle = child_link.object.handle -%}

      {%- if link_collection_handle == collection_handle -%}
        {%- assign should_have_breadcrumb = true -%}
        {%- assign breadcrumb_link = link -%}
        {%- assign child_breadcrumb_link = child_link -%}

        {%- break -%}
      {%- endif -%}

      {%- if child_link.links.size > 0 -%}
        {%- for grandchild_link in child_link.links -%}
          {%- if link.type != 'collection_link' -%}
            {% continue %}
          {%- endif -%}

          {%- assign link_collection_handle = grandchild_link.object.handle -%}

          {%- if link_collection_handle == collection_handle -%}
            {%- assign should_have_breadcrumb = true -%}
            {%- assign breadcrumb_link = link -%}
            {%- assign child_breadcrumb_link = child_link -%}
            {%- assign grandchild_breadcrumb_link = grandchild_link -%}

            {%- break -%}
          {%- endif -%}
        {%- endfor -%}
      {%- endif -%}
    {%- endfor -%}
  {%- endif -%}
{%- endfor -%}

{%- if should_have_breadcrumb -%}
  {%- if breadcrumb_link -%}
    {%- assign breadcrumbs = '|' | append: breadcrumb_link.title | append: ',' | append: breadcrumb_link.url -%}
  {%- endif -%}

  {%- if child_breadcrumb_link -%}
    {%- assign breadcrumbs = breadcrumbs | append: '|' | append: child_breadcrumb_link.title | append: ',' | append: child_breadcrumb_link.url -%}
  {%- endif -%}

  {%- if grandchild_breadcrumb_link -%}
    {%- assign breadcrumbs = breadcrumbs | append: '|' | append: grandchild_breadcrumb_link.title | append: ',' | append: grandchild_breadcrumb_link.url -%}
  {%- endif -%}

  {%- assign breadcrumbs = breadcrumbs | remove_first: '|' | split: '|' -%}
{%- endif -%}
```

## Example Usage

### Breadcrumbs (as per the first image)
```liquid
<nav class="clc-Breadcrumb">
  <div class="clc-Breadcrumb_Inner lyt-Container">
    <div class="clc-Breadcrumb_Body">
      <nav class="bdc-Breadcrumb">
        <div class="bdc-Breadcrumb_Inner">
          <div class="bdc-Breadcrumb_Body">
            <ul class="bdc-Breadcrumb_Items">
              <li class="bdc-Breadcrumb_Item">
                <a class="bdc-Breadcrumb_Link" href="{{ shop.url }}">
                  {{ 'layout.breadcrumbs.home' | t }}
                </a>
              </li>

              {%- for breadcrumb in breadcrumbs -%}
                {%- assign breadcrumb_split = breadcrumb | split: ',' -%}
                {%- assign title = breadcrumb_split[0] -%}
                {%- assign url = breadcrumb_split[1] -%}

                <li class="bdc-Breadcrumb_Item">
                  {%- if forloop.last -%}
                    {{ title }}
                  {%- else -%}
                    <a class="bdc-Breadcrumb_Link"
                       href="{{ url }}">
                      {{ title }}
                    </a>
                  {%- endif -%}
                </li>
              {%- endfor -%}
            </ul>
          </div>
        </div>
      </nav>
    </div>
  </div>
</nav>
```

### Breadcrumbs Schema
```liquid
  <script type="application/ld+json">
    {
      "@context": "http://schema.org/",
      "@type": "BreadcrumbList",
      "itemListElement": [
        {%- for breadcrumb in breadcrumbs -%}
          {%- assign breadcrumb_split = breadcrumb | split: ',' -%}
          {%- capture link -%}{{ shop.url }}/{{ breadcrumb_split[1] }}{%- endcapture -%}
          {%- assign title = breadcrumb_split[0] -%}
          {
            "@type": "ListItem",
            "position": "{{ forloop.index }}",
            "item": {
              "@id": "{{ link }}",
              "name": "{{ title }}"
            }
          }{%- unless forloop.last -%},{%- endunless -%}
        {%- endfor -%}
      ]
    }
  </script>
```
