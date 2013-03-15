---
layout: post
title: "Using MVC4 bundler with a Content Delivery Network"
author: Peter Gledhill
date: 2013-03-15 10:46
comments: true
categories: [C#, MVC4]
---

## Problem

With the MVC4 bundler; you might come to a point where you want to host your assets on a CDN.

Well that's easy hey? The bundler allows you to specify a CDN url for each bundle, so you can use a local file when developing and the file from the CDN when deployed. Like so:

{% codeblock lang:csharp MVC4 Bundling and Minification %}

bundles.UseCdn = true;   //enable CDN support
bundles.Add(
	new ScriptBundle("~/bundles/site-scripts", "http://MYCDNURL/bundles/site-scripts")
	.Include(
    	"~/Scripts/sitejs.js",
    	"~/Scripts/sitejs2.js"
    )
);

// will produce a tags like so:
// in development: <script src="/bundles/site-scripts?v=BAhWhKO17E6GODxEzM0RwLP7V19cP-pbsCGwKj79-EI1" ></script>
// in production : <script src="http://MYCDNURL/bundles/site-scripts" ></script>

{% endcodeblock %}

The problem with this approach is that you lose the cache busting query string which the bundler generates.  The query string is a hash of bundle contents so it changes when the content changes. 

The query string is handy to have when deploying to a CDN as it means you don't have to manually invalidate the file on the CDN (something which I'd quite easily forget to do).

__Note: Amazon CloudFront can be configured to respond to query strings. Some CDNs ignore them.  If that's the case, then this solution won't help.__

__Note: I never actually upload anything to my CDN. I'm have my CDN setup to pull files from my webserver when they are first requested. This is called an 'Origin Pull' setup. __

## Solution

This solution is taken directly form this [stackoverflow answer by Mike Richards](http://stackoverflow.com/a/6892794 "Put images on CDN, using MVC3 on IIS7").

The solution uses an ActionFilter to rewrite the response before it is sent back to the client.  You should end up with scripts / css tags like so: 

{% codeblock lang:html %}
// protocol left off to match the parent request
<script src="//MYCDNURL/bundles/site-scripts?v=BAhWhKO17E6GODxEzM0RwLP7V19cP-pbsCGwKj79-EI1" ></script>
{% endcodeblock %}

First create a custom MemoryStream class to rewrite the stream:
{% gist 5169331 CdnRewriteResponse.cs %}

Create an ActionFilter which passes the response stream to CdnRewriteResponse
{% gist 5169331 CdnRewriteResponseFilter.cs %}

Apply the filter globally
{% gist 5169331 FilterConfig.cs %}

## Thoughts

This is a brute force solution to the problem and it's not that efficient: the whole document has to be parsed to update a few tags.  However, it only seemed to add a few milliseconds to the request and the webserver is freed from serving static files (probably a saving overall). 
