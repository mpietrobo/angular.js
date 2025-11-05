# Security Policy

## Supported Versions

**AngularJS support has officially ended as of January 2022.**
[See what ending support means](https://docs.angularjs.org/misc/version-support-status)
and [read the end of life announcement](https://goo.gle/angularjs-end-of-life).

Visit [angular.io](https://angular.io) for the actively supported Angular.

| Version     | Supported          | Status                | Comments                             |
| ----------- | ------------------ | --------------------- | ------------------------------------ |
| 1.8.x       | :x:                | All support ended     |                                      |
| 1.3.x-1.7.x | :x:                | All support ended     |                                      |
| 1.2.x       | :x:                | All support ended     | Last version to provide IE 8 support |
| <1.2.0      | :x:                | All support ended     |                                      |

## Known CVEs to fix

- CVE-2024-8373 : Content Spoofing

    https://www.herodevs.com/vulnerability-directory/cve-2024-8373?nes-for-angularjs

  Normally, setting an <img> or <source> element's srcset attribute value is subject to image source sanitization, which allows
  improving the security of an application by setting restrictions on the sources of images that can be shown. For example, only
  allowing images from a specific domain.

  However, due to a bug in AngularJS, setting a <source> element’s srcset attribute via the ngAttrSrcset directive or interpolation
  is not subject to image source sanitization. This allows bypassing the image source restrictions configured in the application,
  which can also lead to a form of Content Spoofing.

  Note: The ngSrcset and ngPropSrcset directives are not affected. With these directives, sanitization works as intended

  scenario:
    configure $compileProvider to only allow images from a specific domain.

```
    angular
    .module('app', [])
    .config(['$compileProvider', $compileProvider => {
      $compileProvider.imgSrcSanitizationTrustedUrlList(
          // Only allow images from `angularjs.org`.
          /^https:\/\/angularjs\.org\//);
    }]);
```

  Use a specially-crafted value in the ngAttrSrcset directive on a <source> element to bypass the domain restriction and show an
  image from a disallowed domain.

```
<picture>
  <source ng-attr-srcset="https://angular.dev/favicon.ico" />
  <img src="" />
</picture>
```

```html
<section class="repro-content-section" ng-app="app">
  <h2>Reproduction</h2>
  <p class="repro-images">
    <span class="section-header">
      Examples with <code>ngSrcset</code><br />
      (not vulnerable)
    </span>
    <span>
      Image from <code>angular.dev</code>:<br />
      <b>BLOCKED</b> (Correct ✔️)
    </span>
    <picture>
      <source ng-srcset="https://angular.dev/favicon.ico" />
      <img src="" />
    </picture>
    <span>
      Arbitrary SVG image via data URL:<br />
      <b>BLOCKED</b> (Correct ✔️)
    </span>
    <picture>
      <source ng-srcset="data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAxMDAgMTAwIj48dGV4dCBmb250LXNpemU9IjEwMCIgeT0iMWVtIj7wn5Cx4oCN8J+SuzwvdGV4dD48L3N2Zz4=" />
      <img src="" />
    </picture>
    <span class="section-header">
      Examples with <code>ngPropSrcset</code><br />
      (not vulnerable)
    </span>
    <span>
      Image from <code>angular.dev</code>:<br />
      <b>BLOCKED</b> (Correct ✔️)
    </span>
    <picture>
      <source ng-prop-srcset="'https://angular.dev/favicon.ico'" />
      <img src="" />
    </picture>
    <span>
      Arbitrary SVG image via data URL:<br />
      <b>BLOCKED</b> (Correct ✔️)
    </span>
    <picture>
      <source ng-prop-srcset="'data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAxMDAgMTAwIj48dGV4dCBmb250LXNpemU9IjEwMCIgeT0iMWVtIj7wn5Cx4oCN8J+SuzwvdGV4dD48L3N2Zz4='" />
      <img src="" />
    </picture>
    <span class="section-header">
      Examples with <code>ngAttrSrcset</code><br />
      (vulnerable)
    </span>
    <span>
      Image from <code>angular.dev</code>:<br />
      <b>ALLOWED</b> (Error ❌)
    </span>
    <picture>
      <source ng-attr-srcset="https://angular.dev/favicon.ico" />
      <img src="" />
    </picture>
    <span>
      Arbitrary SVG image via data URL:<br />
      <b>ALLOWED</b> (Error ❌)
    </span>
    <picture>
      <source ng-attr-srcset="data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAxMDAgMTAwIj48dGV4dCBmb250LXNpemU9IjEwMCIgeT0iMWVtIj7wn5Cx4oCN8J+SuzwvdGV4dD48L3N2Zz4=" />
      <img src="" />
    </picture>
    <span class="section-header">
      Examples with <code>srcset</code> interpolation<br />
      (vulnerable)
    </span>
    <span>
      Image from <code>angular.dev</code>:<br />
      <b>ALLOWED</b> (Error ❌)
    </span>
    <picture>
      <source srcset="{{ 'https://angular.dev/favicon.ico' }}" />
      <img src="" />
    </picture>
    <span>
      Arbitrary SVG image via data URL:<br />
      <b>ALLOWED</b> (Error ❌)
    </span>
    <picture>
      <source srcset="{{ 'data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAxMDAgMTAwIj48dGV4dCBmb250LXNpemU9IjEwMCIgeT0iMWVtIj7wn5Cx4oCN8J+SuzwvdGV4dD48L3N2Zz4=' }}" />
      <img src="" />
    </picture>
  </p>
</section>
```


- CVE-2024-8372 : Content Spoofing

    https://www.herodevs.com/vulnerability-directory/cve-2024-8372?nes-for-angularjs

  The logic used in the ngSrcset, ngAttrSrcset and ngPropSrcset directives to sanitize image source URLs has a vulnerability that
  allows bypassing the restrictions set by some common patterns, such as only allowing images from a specific domain. With a
  specially-crafted input, the sanitization can be bypassed and images from an arbitrary domain can be shown, which can also lead
  to a form of Content Spoofing.
  Note: This issue also affects setting interpolated values via the srcset HTML attribute, which is not recommended in AngularJS anyway.

  examples:
  `<img ng-srcset="https://angularjs.org/favicon.ico xyz,https://angular.dev/favicon.ico" />`
  `<img ng-srcset="https://angularjs.org/favicon.ico xyz,data:image/svg+xml;base64,..." />`


- CVE-2025-0716 : Content Spoofing

    https://www.herodevs.com/vulnerability-directory/cve-2025-0716?nes-for-angularjs

  An improper sanitization vulnerability (CVE-2025-0716) has been identified in AngularJS, which allows attackers to bypass common
  image source restrictions normally applied to the value of the href or xlink:href attributes on <image> SVG elements. This bypass
  can further lead to a form of Content Spoofing. Similarly, the application's performance and behavior could be negatively affected
  by using too large or slow-to-load images.

  Normally, setting an <image> SVG element's href or xlink:href attribute values via AngularJS bindings is subject to image source
  sanitization. This allows improving the security of an application by setting restrictions on the sources of images that can be
  shown. For example, only allowing images from a specific domain.

  However, due to a bug in AngularJS, setting an <image> SVG element's href or xlink:href attribute values via the ngHref and
  ngAttrHref directives or using interpolation is not subject to image source sanitization. This allows bypassing the image source
  restrictions configured in the application, which can also lead to a form of Content Spoofing. Similarly, the application's
  performance and behavior can be negatively affected by using too large or slow-to-load images.

  Note: Targeting the xlink:href attribute via ng-attr-xlink:href or interpolation is not affected. With xlink:href, sanitization
  works as intended.


- CVE-2025-2336 : Content Spoofing

    https://www.herodevs.com/vulnerability-directory/cve-2025-2336?nes-for-angularjs

  An improper sanitization vulnerability (CVE-2025-2336) has been identified in AngularJS' ngSanitize module, which allows attackers
  to bypass common image source restrictions normally applied to image elements. This bypass can further lead to a form of Content
  Spoofing. Similarly, the application's performance and behavior could be negatively affected by using too large or slow-to-load
  images.

  The $sanitize service, which is provided by the angular-sanitize package, is used for sanitizing HTML strings by stripping all
  potentially dangerous tokens. As part of the sanitization, it checks the URLs of images to ensure they abide by the defined image
  source rules. This allows improving the security of an application by setting restrictions on the sources of images that can be
  shown. For example, only allowing images from a specific domain.

  However, due to a bug in the $sanitize service, SVG <image> elements are not correctly detected as images, even when SVG support
  is enabled. As a result, the image source restrictions are not applied to the images that can be shown. This allows bypassing the
  image source restrictions configured in the application, which can also lead to a form of Content Spoofing. Similarly, the
  application's performance and behavior can be negatively affected by using too large or slow-to-load images.

  Note: The $sanitize service is also internally used by the ngBindHtml directive and the linky filter, so any vulnerabilities
  affect them as well.

 example:

```
   angular
    .module('app', ['ngSanitize'])
    .config([
      '$compileProvider', '$sanitizeProvider',
      ($compileProvider, $sanitizeProvider) => {
        $compileProvider.imgSrcSanitizationTrustedUrlList(
            // Only allow images from `angularjs.org`.
            /^https:\/\/angularjs\.org\//);

        // Enable SVG support in `$sanitize()`.
        $sanitizeProvider.enableSvg(true);
      },
    ]);
```

```
<svg>
  <image href="https://angular.dev/favicon.ico"></image>
  <!--
    OR:
    <image xlink:href="https://angular.dev/favicon.ico"></image>
  -->
</svg>
```

- CVE-2025-4690 : Regular Expression Denial of Service

    https://www.herodevs.com/vulnerability-directory/cve-2025-4690?nes-for-angularjs

  The linky filter, which is provided by the angular-sanitize package, is used for finding links in an input text and turning them
  into sanitized HTML links (using the $sanitize service under the hood). The logic for finding links in text is powered by a
  Regular Expression.

  Due to an implementation bug, the Regular Expression has a super-linear runtime relative to the input size. With a long,
  specially-crafted input, an attacker could cause a denial of service of the application, monopolizing browser resources or
  completely crash the application.


- CVE-2022-37602 : Prototype Pollution [ IGNORE - N/A ] (this affects grunt builds, not build artefacts)

    https://www.herodevs.com/vulnerability-directory/cve-2022-37602?nes-for-angularjs

  A Prototype Pollution vulnerability (CVE-2022-37602) has been identified in grunt-karma, which allows a malicious actor to modify
  an object's prototype, potentially leading to unexpected behavior or security issues.


## CVEs fixed

- CVE-2022-25844 : ReDoS Vulnerability [FIXED]

    https://www.herodevs.com/vulnerability-directory/cve-2022-25844?nes-for-angularjs

  AngularJS' localization utilities allow formatting numbers as currency values. If an application sets the current locale's
  NUMBER_FORMATS.PATTERNS[1].posPre value to a user-defined string, it can be abused to cause a Denial of Service of the appication.


- CVE-2023-26116 : ReDoS Vulnerability [FIXED]

    https://nvd.nist.gov/vuln/detail/CVE-2023-26116
    https://www.herodevs.com/vulnerability-directory/cve-2023-26116?nes-for-angularjs

  It’s possible in versions of Angular starting from 1.2.21 to conduct a Regular Expression Denial of Service (ReDoS) attack via the
  angular.copy() utility function. If a malicious actor carefully composes an insecure regular expression and provides it to the
  function, it can cause catastrophic backtracking and monopolize system resources. A proof of concept demonstrating this exploit is
  available on StackBlitz.

    equivalent to: `angular.copy(new RegExp(userProvidedPattern));`


- CVE-2023-26117 : ReDoS Vulnerability [FIXED]

    https://nvd.nist.gov/vuln/detail/CVE-2023-26117
    https://www.herodevs.com/vulnerability-directory/cve-2023-26117?nes-for-angularjs

  It’s possible in versions of Angular starting from 1.0.0 to conduct a Regular Expression Denial of Service (ReDoS) attack via the
  $resource service due to the usage of an insecure regular expression. If a malicious actor carefully composes an insecure resource URL
  value and provides it to the service, it can cause catastrophic backtracking and monopolize system resources.

  A regular expression used by the $resource service to strip trailing slashes is vulnerable to super-linear runtime due to
  backtracking. With a large carefully-crafted input, this can result in catastrophic backtracking and cause a denial of service of
  the application

    equivalent to `$resource(`/some/url/${userProvidedSuffix}`).query()`


- CVE-2023-26118 : ReDoS Vulnerability [FIXED]

    https://nvd.nist.gov/vuln/detail/CVE-2023-26118
    https://www.herodevs.com/vulnerability-directory/cve-2023-26118?nes-for-angularjs

  It’s possible in versions of Angular starting from 1.4.9 to conduct a Regular Expression Denial of Service (ReDoS) attack via the
  <input type="url"> element. If a malicious actor carefully composes an insecure URL input that is used by the input[url] element,
  catastrophic backtracking and monopolization of system resources can occur.

  A regular expression used to validate the value of the input[url] directive is vulnerable to super-linear runtime due to backtracking


- CVE-2024-21490 : ReDoS Vulnerability [FIXED]

    https://nvd.nist.gov/vuln/detail/CVE-2024-21490
    https://www.herodevs.com/vulnerability-directory/cve-2024-21490?nes-for-angularjs

  Starting with version 1.3.0 of Angular, it’s possible to conduct a Regular Expression Denial of Service (ReDoS) attack. Because the
  package uses a regular expression to split the value of the ng-srcset directive, if a malicious actor carefully composes an
  ng-scset value, this can cause catastrophic backtracking and monopolize system resources


## CVEs ignored because not applicable to our use-cases

- CVE-2022-25869 : Cross-Site Scripting [ IGNORED - IE NOT SUPPORTED ] ( we do not support anymore IE )

    https://nvd.nist.gov/vuln/detail/CVE-2022-25869
    https://www.herodevs.com/vulnerability-directory/cve-2022-25869?nes-for-angularjs

  This Cross-Site Scripting (XSS) exploit is present in all public versions of AngularJS. It is present only with the Internet
  Explorer browser, which has a bug in its page caching when dealing with textareas. A malicious actor can insert dangerous code
  that the browser will execute thereby giving access to data or script function (the attacker tricks the application or site into
  accepting a request as though it was from a trusted source).


- CVE-2024-33665 : Cross-Site Scripting [IGNORED - N/A ] (we do not use angular-translate)

    https://www.herodevs.com/vulnerability-directory/cve-2024-33665?nes-for-angularjs

  The vulnerability can be triggered by injecting malicious code into input fields that are then processed by the translate directive
