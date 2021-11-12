# plugin-structured-data
A plugin for [Micro.blog](https://micro.blog "Micro.blog") that injects [structured data](https://developers.google.com/search/docs/advanced/structured-data/intro-structured-data "Structured Data Intro") for blog posts via a JSON-LD script tag. This is what search engines use to decorate their results. Confirmation that the structured data is accessible can be had by using Google’s [rich results test](https://search.google.com/test/rich-results "Rich Results").

## Parameters
![Plugin Parameters](https://raw.githubusercontent.com/moonbuck/plugin-structured-data/main/plugin_parameters.jpeg)

You can enter an `Author Name` here, the plugin falls back to `site.Author.name` and `site.Params.Author.name`.

The `Author Profile URL` should point to an about or profile page. If you don’t have one you can leave this empty.

## Files
There is only one file. It lives at `layouts/partials/structured-data-injection.html` and it looks like this:

```go
{{- if (eq .Type "post") -}}

{{- /* Create a Scratch instance to gather data */ -}}
{{- $data := newScratch -}}

{{- /* Set the context and type */ -}}
{{- $data.SetInMap "data" "@context" "https://schema.org" -}}
{{- $data.SetInMap "data" "@type" "BlogPosting" -}}

{{- /* Add the author */ -}}
{{- $author_name := (site.Author.name | default site.Params.Author.name) | default site.Params.structured_data_author_name -}}
{{- with $author_name -}}
{{- $data.SetInMap "author" "@type" "Person" -}}
{{- $data.SetInMap "author" "name" . -}}
{{- with site.Params.structured_data_author_profile_url -}}
{{- $data.SetInMap "author" "url" . -}}
{{- end -}}
{{- end -}}
{{- with $data.Get "author" -}}
{{- $data.SetInMap "data" "author" . -}}
{{- end -}}

{{- /* Add the published and modified times */ -}}
{{- $iso8601 := site.Params.Theme.Date.ISO8601 -}}
{{- $date_published := cond .PublishDate.IsZero .Date .PublishDate -}}
{{- if not $date_published.IsZero -}}
{{- $date_published = $date_published.Format $iso8601 -}}
{{- else -}}
{{- $date_published = nil -}}
{{- end -}}
{{- with $date_published -}}
{{- $data.SetInMap "data" "datePublished" . -}}
{{- end -}}
{{- $date_modified := .Lastmod -}}
{{- if not $date_modified.IsZero -}}
{{- $date_modified = $date_modified.Format $iso8601 -}}
{{- else -}}
{{- $date_modified = nil -}}
{{- end -}}
{{- with $date_modified -}}
{{- $data.SetInMap "data" "dateModified" . -}}
{{- end -}}

{{- /* Add the title */ -}}
{{- with .Title -}}
{{- $data.SetInMap "data" "headline" . -}}
{{- end -}}

{{- /* Add any images */ -}}
{{- with .Params.images -}}
{{- $data.SetInMap "data" "image" (apply . "absURL" ".") -}}
{{- end -}}

{{- /* Add description */ -}}
{{- with .Summary -}}
{{- $data.SetInMap "data" "description" (. | plainify | chomp) -}}
{{- end -}}

{{- /* Add the word count */ -}}
{{- with .WordCount -}}
{{- $data.SetInMap "data" "wordcount" . -}}
{{- end -}}

{{- $data = $data.Get "data" -}}

<script type="application/ld+json">{{ $data | jsonify | safeHTML }}</script>

{{- end -}}
```

## Example Output
As an example, the JSON object the plugin generates for [this blog post](https://moondeer-test.micro.blog/2021/11/11/feeding-data-to.html "Feeding Data to My Plugins") looks like this:

```json
{
  "@context": "https://schema.org",
  "@type": "BlogPosting",
  "author": {
    "@type": "Person",
    "name": "Jason Cardwell",
    "url": "https://moondeer.blog/about/"
  },
  "dateModified": "2021-11-11T09:55:43-08:00",
  "datePublished": "2021-11-11T09:55:43-08:00",
  "description": "A guide to navigating the non-obvious nature of the process for providing the data my plugins want … and that you want to give them.",
  "headline": "Feeding Data to My Plugins",
  "image": [
    "https://moondeer.blog/uploads/2021/37f6e3a16c.jpg",
    "https://moondeer.blog/uploads/2021/9e5b65df2c.jpg",
    "https://moondeer.blog/uploads/2021/3de49bd735.jpg",
    "https://moondeer.blog/uploads/2021/cdde25d6af.jpg",
    "https://moondeer.blog/uploads/2021/8d23292ddb.jpg",
    "https://moondeer.blog/uploads/2021/b61c018a99.jpg",
    "https://moondeer.blog/uploads/2021/10b51fae7c.jpg",
    "https://moondeer.blog/uploads/2021/d521640777.jpg",
    "https://moondeer.blog/uploads/2021/f1dca97623.jpg",
    "https://moondeer.blog/uploads/2021/594929f04c.jpg",
    "https://moondeer.blog/uploads/2021/b4f2efbfdc.jpg",
    "https://moondeer.blog/uploads/2021/054aa61bc8.jpg",
    "https://moondeer.blog/uploads/2021/d35974d41e.jpg",
    "https://moondeer.blog/uploads/2021/8bfa3ab25e.jpg",
    "https://moondeer.blog/uploads/2021/633acc7df9.jpg",
    "https://moondeer.blog/uploads/2021/b40d718c91.jpg",
    "https://moondeer.blog/uploads/2021/4e2ba2b2e5.jpg",
    "https://moondeer.blog/uploads/2021/0b3b916c4f.jpg",
    "https://moondeer.blog/uploads/2021/07b83ae2e4.jpg",
    "https://moondeer.blog/uploads/2021/eccab67f51.jpg"
  ],
  "wordcount": 3268
}
```
