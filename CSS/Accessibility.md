Instead of just using divs, we should try to make the html as semantic as possible, by semantic we mean using the correct html elements.

---

#### When to use header
It's a container used to define header for a document or a section.
Headers should have headings h1-h6 and some p tags within them.


The difference between heading and div is that for heading tags, there are margins around it, so we will see empty space. Not with div.

```html
<header>
	<h3 style="color: green;">
	  GeeksforGeeks Learning
	</h3>
	<p>
		A Computer Science portal for geeks.
		It contains well written, well thought <br>
		and well explained computer science and
		programming articles.
	</p>
</header>
```


#### When to use nav?
Nav is a container that identifies a group of navigation links.
E.g. menus/table of contents.

Note `li` elements should always be under `ol` or `ul`
```html
<nav class="crumbs">
  <ol>
    <li class="crumb"><a href="#">Bikes</a></li>
    <li class="crumb"><a href="#">BMX</a></li>
    <li class="crumb">Jump Bike 3000</li>
  </ol>
</nav>
```


#### When to use section?
It represents a section of the document, used when there isn't a more specific semantic element to represent it. Should should always have a heading, with very few exceptions.

We can use it to group related things.

```html
/* before */
<div>
  <h2>Heading</h2>
  <p>Bunch of awesome content</p>
</div>

/* after */
<section>
  <h2>Heading</h2>
  <p>Bunch of awesome content</p>
</section>
```


#### When to use aside?
Used to identify contents related to the primary content of the webpage, but does not constitute the primary content. E.g. Author info, related links, related content, ads are examples.

Things like article data, and post specific navigation (like next post or previous post) are not good uses of `aside` elements.

```html
<main>
	<!-- Article Content -->
	<aside><!-- Pull Quote --></aside>
</main>
	
<aside>
	<!-- Comment Section -->
</aside>
```

#### When to use article?
Self contained piece of content.

On a page with a single piece of content, a single article can be used to contain the main content.

On a page with multiple pieces of content, E.g. a search results page, a news feed. Multiple article elements can be used to contain each individual piece.


Use article when you want to represent independent, self contained pieces that can be used on their own, without any other context.

Use section when you want to group content semantically based on a thematic grouping.

```html
<article class="forecast">
	<h1>Weather forecast for Seattle</h1>
	
	<article class="day-forecast">
		<h2>03 March 2018</h2>
		<p>Rain.</p>
	</article>
	
	<article class="day-forecast">
		<h2>04 March 2018</h2>
		<p>Periods of rain.</p>
	</article>
</article>
```




