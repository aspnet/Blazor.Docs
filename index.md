---
title: Welcome to Blazor Docs website!
---
<style type="text/css">
/* Colors */

.container {
    width: 100%;
    padding: 0;
}

.article
{
  margin-top: 0;
}

.article > .col-md-10
{
    width: 100%;
}

.article > .col-md-2
{
    display: none;
}

div.subnav.navbar.navbar-default
{
  display: none;
}

button, a, .btn-primary {
  color: #47A7A0;
}
.btn-primary:hover, .btn-primary:focus, button:hover, button:focus, a:hover, a:focus, .toc .nav > li.active > a:hover, .toc .nav > li.active > a:focus, .toc .nav > li > a:hover, .toc .nav > li > a:focus {
  color: #6E8B60;
}
.btn-primary.disable, .btn-primary.disable:hover, a.disable, a.disable:hover{
  color: #90ABB9;
}
.hero {
    color: #C1D38B;
    background-color: #385361;
    background-image: -webkit-gradient(linear,left bottom,right top,color-stop(0%,#000),color-stop(100%,#385361));
    background-image: -webkit-linear-gradient(180deg,#000 0,#803612 100%);
    background-image: -moz-linear-gradient(180deg,#000 0,#803612 100%);
    background-image: -ms-linear-gradient(180deg,#000 0,#803612 100%);
    background-image: -o-linear-gradient(180deg,#000 0,#803612 100%);
    background-image: linear-gradient(180deg,#000 0,#803612 100%);
    filter: progid:DXImageTransform.Microsoft.gradient(startColorstr='#000', endColorstr='#803612', GradientType=1);

    margin-top: -50px;
}
.hero strong {
    color: #e7e7e7;
}
.buttons-unit .button {
    color: #fff;
    background: #6E8B60;
}
.buttons-unit .button:active {
    background: #A27865;
}
.toc .nav > li.active > a, .affix ul > li.active > a, .affix ul > li.active > a:before {
  color: #6E8B60
}
.btn-primary {
    background: #385361;
    color: #8BFEB4;
}

.affix ul > li > a:before {
  color: #cccccc;
}
.toc .nav > li > a, .affix ul > li > a, .affix ul > li > a:hover {
  color: #666666;
}

svg:hover path {
    fill: #ffffff;
}

.counter-key-section{
  border: 2px solid #6E8B60;
  -webkit-border-image-source: -webkit-gradient(linear, left, right, color-stop(0%, rgba(0, 0, 0, 0)), color-stop(50%, #385361), color-stop(100%, rgba(0, 0, 0, 0)));
  -webkit-border-image-source: -webkit-linear-gradient(90deg, rgba(0, 0, 0, 0), #385361 50%, rgba(0, 0, 0, 0));
  -moz-border-image-source: -moz-linear-gradient(90deg, rgba(0, 0, 0, 0), #385361 50%, rgba(0, 0, 0, 0));
  -ms-border-image-source: -ms-linear-gradient(90deg, rgba(0, 0, 0, 0), #385361 50%, rgba(0, 0, 0, 0));
  -o-border-image-source: -o-linear-gradient(90deg, rgba(0, 0, 0, 0), #385361 50%, rgba(0, 0, 0, 0));
  border-image-source: linear-gradient(90deg, rgba(0, 0, 0, 0), #385361 50%, rgba(0, 0, 0, 0));
  border-image-slice: 1;
}

/* End Colors */

.value-prop-heading {
  font-size: 24px;
}

#vp-container{
  margin-top: 30px;
}

.hero {
    height: 350px;
    margin-top: 50px;
    padding-top: 50px;
    font-weight: 300;
    text-align: center;
}
.key-section{
  padding: 30px 0;
}

.key-section .glyphicon, .counter-key-section .glyphicon {
  font-size: 8em;
  padding: 50px;
  display: table-cell;
  vertical-align: middle;
}
.glyphicon {
  margin-right: 10px;
  font-size: 16px;
}

.key-section section, .counter-key-section section {
  display: table-cell;
  padding: 40px;
}

.key-section section p, .counter-key-section section p {
  text-align: initial;
}

.counter-key-section{
  padding: 30px 0;
}
.buttons-unit-small{
  font-size: 16px;
}
.version-link, .github-link{
  margin: 5px;
}
.hero strong {
    font-weight: 400;
    font-family: Rockwell;
}
.hero .text {
    font-size: 64px;
    text-align: center;
}
.hero .minitext {
    font-size: 20px;
    text-align: center;
    font-family: Candara;
}
.buttons-unit {
    margin-top: 60px;
    text-align: center;
}
.hero .button {
    border-radius: 4px;
    padding: 8px 16px;
    margin: 0 12px;
    box-shadow: 1px 3px 3px rgba(0,0,0,0.3);
}
.buttons-unit .button {
    font-size: 24px;
}

@media only screen and (max-width: 768px) {
  .hero {
    height: 500px;
    margin-top: 0px;
  }
  .hero .button {
    display: block;
    margin: 12px;
  }
}

footer{
  position: relative;
}
</style>

<div class="hero">
  <div class="wrap">
    <div class="text">
      <strong>BLAZOR</strong>
    </div>
    <div class="buttons-unit-small">
      <a class="version-link" href="#">Version Notes</a><span>|</span><a class="github-link" href="https://github.com/dotnet/docfx">View in Github</a>
    </div>
    <div class="minitext">
        Blazor is an experimental .NET web framework using C#/Razor and HTML that runs in the browser with WebAssembly
    </div>
    <div class="buttons-unit">
      <a href="#" class="button"><i class="glyphicon glyphicon-send"></i>Get Started</a>
    </div>
  </div>
</div>
<div class="key-section">
  <div class="container">
    <div class="row">
      <div class="col-md-8 col-md-offset-2 text-center">
        <i class="glyphicon glyphicon-wrench"></i>
        <section>
          <h2>Build Web UI with C#</h2>
          <p class="lead">Blazor is a web UI framework, similar to existing JavaScript frameworks like Angular or React, but you write C# instead of JavaScript.</p>
          <a href="https://github.com/aspnet/Blazor/wiki/FAQ" class="btn btn-primary">What is Blazor</a>
        </section>
      </div>
    </div>
  </div>
</div>
<div class="counter-key-section">
  <div class="container">
    <div class="row">
      <div class="col-md-8 col-md-offset-2 text-center">
        <section>
          <h2>Full Stack .NET</h2>
          <p class="lead">Do full stack .NET development using stable and consistent tools, languages and APIs both in the browser and on the server. </p>
          <a href="https://www.microsoft.com/net" class="btn btn-primary">Learn more about the .NET platform</a>
        </section>
        <i class="glyphicon glyphicon-tasks"></i>
      </div>
    </div>
  </div>
</div>
<div class="key-section">
  <div class="container content">
    <div class="row">
      <div class="col-md-8 col-md-offset-2 text-center">
        <i class="glyphicon glyphicon-globe"></i>
        <section>
          <h2>.NET Standard, Works in all browsers</h2>
          <p class="lead">Blazor runs in all browsers, on the real .NET runtime with full support for .NET Standard (i) [info bubble that explains what .NET Standard is]. Blazor requires no plugins and no code transpilation, only open web standards. </p>
          <a href="http://blog.stevensanderson.com/2018/02/06/blazor-intro/" class="btn btn-primary">How Blazor works</a>
        </section>
      </div>
    </div>
</div>
<div class="counter-key-section">
  <div class="container">
    <div class="row">
      <div class="col-md-8 col-md-offset-2 text-center">
        <section>
          <h2>Native performance with WebAssembly </h2>
          <p class="lead">Runs on WebAssembly, giving you native performance in the browser and a trusted security sandbox. </p>
        </section>
        <i class="glyphicon glyphicon-fire"></i>
      </div>
    </div>
  </div>
</div>
<div class="key-section">
  <div class="container content">
    <div class="row">
      <div class="col-md-8 col-md-offset-2 text-center">
        <i class="glyphicon glyphicon-transfer"></i>
        <section>
          <h2>Native browser apps </h2>
          <p class="lead">Easily interact with your existing JavaScript code, your favorite libraries, and any browser API through JavaScript interop. </p>
        </section>
      </div>
    </div>
</div>
<div class="counter-key-section">
  <div class="container">
    <div class="row">
      <div class="col-md-8 col-md-offset-2 text-center">
        <section>
          <h2>Simple and productive </h2>
          <p class="lead">Get started fast and remain productive with project templates, great tooling, and reusable components.</p>
          <a href="https://marketplace.visualstudio.com/items?itemName=aspnet.blazor" class="btn btn-primary">Tools for Blazor</a>
        </section>
        <i class="glyphicon glyphicon-console"></i>
      </div>
    </div>
  </div>
</div>
<div class="key-section">
  <div class="container content">
    <div class="row">
      <div class="col-md-8 col-md-offset-2 text-center">
        <i class="glyphicon glyphicon-user"></i>
        <section>
          <h2>Get involved </h2>
          <p class="lead">Join the communit that's building Blazor, writing documentation, building samples, and more!</p>
        </section>
      </div>
    </div>
</div>
<div class="counter-key-section">
  <div class="container">
    <div class="row">
      <div class="col-md-8 col-md-offset-2 text-center">
        <section>
          <h2>Open-source & free </h2>
          <p class="lead">Blazor and the .NET platform are open-source, with a strong community of 25,000+ contributors from over 1,700 companies. </p>
          <a href="https://github.com/aspnet/blazor" class="btn btn-primary">Blazor on GitHub</a>
        </section>
        <i class="glyphicon glyphicon-road"></i>
      </div>
    </div>
  </div>
</div>