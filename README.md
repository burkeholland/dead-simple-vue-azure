# Dead simple Vue static site on Azure AppService

This is a sample project to show you how you can run Vue as a static site on [Azure AppService](https://code.visualstudio.com/tutorials/app-service-extension/getting-started?WT.mc_id=m365-0000-buhollan). Azure AppService offers both Linux and Windows hosting, so this project covers both.

## Linux

Node on Azure AppService for Linux does not come with a web server. This means we need to bring one of our own. The easiest way to do that is to use a Node web server like [serve](https://www.npmjs.com/package/serve). Azure uses [pm2](http://pm2.keymetrics.io/) behind the scenes, so we can start "serve" with the simplest possible `ecosystem.config.js` file. The "-s" flag makes sure that single page application routing works.

```
module.exports = {
  apps: [
    {
      script: "npx serve -s"
    }
  ]
};
```

This file needs to be in the "dist" folder after a build, so put them in the "public" folder in your Vue project. Anything in the "public" folder gets copied directly to the root of "dist".

Now deploy the "dist" folder to Azure AppService.

## Windows

Windows hosting has a built-in web server (IIS), so your project will "just work" when you deploy it. Kind of.

You will need to inform IIS to route all traffic to our one `index.html` page. This requires a `web.config` file.

```
<?xml version="1.0"?>
<configuration>
  <system.webServer>
    <rewrite>
      <rules>
        <rule name="Vue" stopProcessing="true">
          <match url=".*" />
          <conditions logicalGrouping="MatchAll">
            <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
          </conditions>
          <action type="Rewrite" url="/" />
        </rule>
      </rules>
    </rewrite>
  </system.webServer>
</configuration>
```

That says route all traffic to `index.html` UNLESS the request is for a file. We need that exception in there because `index.html` is a file and we need it to work.

We want this `web.config` file to be in the root of the "dist" folder, so we can just put it in the "public" folder in our Vue project and check it right into source control.

That's it!
