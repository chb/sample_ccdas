# Kinsights C-CDA Sample

Kinsights allows its users to export their records via a [bbClear](https://blue-button.github.io/bbClear/) HTML document, which contains the raw data in the form of an embedded C-CDA XML document.

The basic structure of the bbClear document is:

    <html>
    <head>...</head>
    <body>...</body>
    </html>
    <script style="display: none;" id="xmlBBData" type="text/plain">
        {{CCDA XML HERE}}
    </script>

The XML data is extractable by the end user in-browser, and this XML file is included in this directory as well.

We hope other C-CDA parsers will eventually recognize the more patient-friendly bbClear wrapper and allow it as an acceptable format for C-CDA importing when the end-user is offered the option to upload C-CDA data.
