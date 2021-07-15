# cloudflare-dropbox-cdn

This is an output from ac-bundle-app.  
You can use the script as Cloudflare worker to use Dropbox as CDN.

```
appConfig = {
  cloudflare: {
    account: "<CLOUDFLARE-ACCOUNT-KEY>",
    email: "<CLOUDFLARE-EMAIL>",
    auth: "<CLOUDFLARE-AUTH-KEY>"
  },
  dropbox: {
    link: "https://content.dropboxapi.com/2/files/download",
    prefix: "/publish",
    accessToken:
      "<YOUR-ACCESS-TOKEN>"
  }
};
var app = {
  start: function () {},
  indexDynamic: async function (request) {
    var url = new URL(request.url);
    if (url.pathname.split("/").pop().trim() === "")
      url.pathname += "index.html";
    var result = await app.api.dropbox.file(url.pathname);
    return app.response(await result.arrayBuffer(), {
      status: result.status,
      headers: {
        "Content-Type": app.utils.mime.get(url.pathname)
      }
    });
  }
};
app.startUps = [];
app.callbacks = { static: [] };
app["api"] = {
  dropbox: (function () {
    var mod = {
      fileCallback: async function (callback, errorCallback, path) {
        path = appConfig.dropbox.prefix + path;
        var result = await fetch(appConfig.dropbox.link, {
          method: "POST",
          headers: {
            Authorization: "Bearer " + appConfig.dropbox.accessToken,
            "Dropbox-API-Arg": JSON.stringify({ path: path })
          }
        });
        await callback(result);
      }
    };
    return mod;
  })()
};
app["build"] = {};
app["publish"] = {};
app["utils"] = {
  mime: (function () {
    var mod = {
      list: {
        aac: "audio/aac",
        abw: "application/x-abiword",
        ai: "application/postscript",
        arc: "application/octet-stream",
        avi: "video/x-msvideo",
        azw: "application/vnd.amazon.ebook",
        bin: "application/octet-stream",
        bz: "application/x-bzip",
        bz2: "application/x-bzip2",
        csh: "application/x-csh",
        css: "text/css",
        csv: "text/csv",
        doc: "application/msword",
        dll: "application/octet-stream",
        eot: "application/vnd.ms-fontobject",
        epub: "application/epub+zip",
        gif: "image/gif",
        htm: "text/html",
        html: "text/html",
        ico: "image/x-icon",
        ics: "text/calendar",
        jar: "application/java-archive",
        jpeg: "image/jpeg",
        jpg: "image/jpeg",
        js: "application/javascript",
        json: "application/json",
        mid: "audio/midi",
        midi: "audio/midi",
        mp2: "audio/mpeg",
        mp3: "audio/mpeg",
        mp4: "video/mp4",
        mpa: "video/mpeg",
        mpe: "video/mpeg",
        mpeg: "video/mpeg",
        mpkg: "application/vnd.apple.installer+xml",
        odp: "application/vnd.oasis.opendocument.presentation",
        ods: "application/vnd.oasis.opendocument.spreadsheet",
        odt: "application/vnd.oasis.opendocument.text",
        oga: "audio/ogg",
        ogv: "video/ogg",
        ogx: "application/ogg",
        otf: "font/otf",
        png: "image/png",
        pdf: "application/pdf",
        ppt: "application/vnd.ms-powerpoint",
        rar: "application/x-rar-compressed",
        rtf: "application/rtf",
        sh: "application/x-sh",
        svg: "image/svg+xml",
        swf: "application/x-shockwave-flash",
        tar: "application/x-tar",
        tif: "image/tiff",
        tiff: "image/tiff",
        ts: "application/typescript",
        ttf: "font/ttf",
        txt: "text/plain",
        vsd: "application/vnd.visio",
        wav: "audio/x-wav",
        weba: "audio/webm",
        webm: "video/webm",
        webp: "image/webp",
        woff: "font/woff",
        woff2: "font/woff2",
        xhtml: "application/xhtml+xml",
        xls: "application/vnd.ms-excel",
        xlsx: "application/vnd.ms-excel",
        xlsx_OLD:
          "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet",
        xml: "application/xml",
        xul: "application/vnd.mozilla.xul+xml",
        zip: "application/zip",
        "3gp": "video/3gpp",
        "3gp_DOES_NOT_CONTAIN_VIDEO": "audio/3gpp",
        "3gp2": "video/3gpp2",
        "3gp2_DOES_NOT_CONTAIN_VIDEO": "audio/3gpp2",
        "7z": "application/x-7z-compressed"
      },
      get: function (ext) {
        ext = ext.split(".").pop().toLowerCase();
        return mod.list[ext];
      }
    };
    return mod;
  })()
};
var config = app.config;
var modules = app.modules;
app.has = function (value) {
  var found = true;
  for (var i = 0; i <= arguments.length - 1; i++) {
    var value = arguments[i];
    if (!(typeof value !== "undefined" && value !== null && value !== ""))
      found = false;
  }
  return found;
};
app.response = function (data, options) {
  if (typeof options === "undefined") options = {};
  if (typeof options.headers === "undefined") options.headers = {};
  options.headers["Access-Control-Allow-Origin"] = "*";
  options.headers["Access-Control-Allow-Headers"] = "content-type";
  options.headers["Access-Control-Allow-Methods"] =
    "HEAD, GET, PUT, DELETE, POST, OPTIONS";
  if (typeof Response !== "undefined") {
    return new Response(data, options);
  } else {
    return { data: data, options: options };
  }
};
(app.logAndResponse = function (data, options) {
  console.log(data);
  return app.response(data, options);
}),
  (app.camelCase = function camelize(str, capitalFirst) {
    if (!app.has(capitalFirst)) capitalFirst = false;
    var result = str
      .replace(/(?:^\w|[A-Z]|\b\w)/g, function (word, index) {
        return index === 0 ? word.toLowerCase() : word.toUpperCase();
      })
      .replace(/\s+/g, "");
    if (capitalFirst)
      result = result.substr(0, 1).toUpperCase() + result.substr(1, 999);
    return result;
  });
app.properCase = function (str) {
  return str.replace(/\w\S*/g, function (txt) {
    return txt.charAt(0).toUpperCase() + txt.substr(1).toLowerCase();
  });
};
if (app.has(app.api) === true) {
  // callbacks
  (function () {
    var callbackLevel = function (apiLevel) {
      if (app.has(apiLevel) && !app.has(apiLevel.length)) {
        for (var moduleName in apiLevel) {
          if (app.has(apiLevel[moduleName]) === true) {
            callbackLevel(apiLevel[moduleName]);
            for (var key in apiLevel[moduleName]) {
              (function (moduleName, key) {
                var func = apiLevel[moduleName][key];
                if (
                  key.split("Callback").length > 1 &&
                  typeof func === "function"
                ) {
                  apiLevel[moduleName][
                    key.split("Callback").shift() + "Multi"
                  ] = async function (
                    count,
                    name,
                    callback,
                    arg1,
                    arg2,
                    arg3,
                    arg4,
                    arg5,
                    arg6,
                    arg7,
                    arg8,
                    arg9,
                    arg10,
                    arg11,
                    arg12,
                    arg13,
                    arg14,
                    arg15
                  ) {
                    return new Promise(function (resolve, reject) {
                      if (!app.has(count)) count = 1;
                      var rCount = 0;
                      var resolveCount;
                      for (var i = 0; i <= count - 1; i++) {
                        (async function (index) {
                          var countResult = await apiLevel[moduleName][
                            key.split("Callback").shift()
                          ](
                            name,
                            async function (arg1, arg2, arg3, arg4, arg5) {
                              if (typeof callback === "function") {
                                var result = await callback(
                                  arg1,
                                  arg2,
                                  arg3,
                                  arg4,
                                  arg5
                                );
                                if (result === true && !app.has(resolveCount)) {
                                  console.log("MULTI INDEX:", index);
                                  resolveCount = index;
                                }
                                return result;
                              }
                            },
                            arg1,
                            arg2,
                            arg3,
                            arg4,
                            arg5,
                            arg6,
                            arg7,
                            arg8,
                            arg9,
                            arg10,
                            arg11,
                            arg12,
                            arg13,
                            arg14,
                            arg15
                          );
                          rCount += 1;
                          if (resolveCount === index || rCount >= count)
                            resolve(countResult);
                        })(i);
                      }
                    });
                  };
                  apiLevel[moduleName][
                    key.split("Callback").shift()
                  ] = async function (
                    name,
                    callback,
                    arg1,
                    arg2,
                    arg3,
                    arg4,
                    arg5,
                    arg6,
                    arg7,
                    arg8,
                    arg9,
                    arg10,
                    arg11,
                    arg12,
                    arg13,
                    arg14,
                    arg15
                  ) {
                    if (typeof callback !== "function") {
                      arg15 = arg13;
                      arg14 = arg12;
                      arg13 = arg11;
                      arg12 = arg10;
                      arg11 = arg9;
                      arg10 = arg8;
                      arg9 = arg7;
                      arg8 = arg6;
                      arg7 = arg5;
                      arg6 = arg4;
                      arg5 = arg3;
                      arg4 = arg2;
                      arg3 = arg1;
                      arg2 = callback;
                      arg1 = name;
                    }
                    var output, error;
                    await apiLevel[moduleName][key](
                      async function (data, page) {
                        var result =
                          typeof callback === "function"
                            ? await callback(data, page)
                            : undefined;
                        if (app.has(result) && app.has(result.length)) {
                          if (!app.has(output)) output = [];
                          output = output.concat(result);
                        } else {
                          output = data;
                        }
                        return result;
                      },
                      function (err, errorText) {
                        error = { error: err, errorText };
                      },
                      arg1,
                      arg2,
                      arg3,
                      arg4,
                      arg5,
                      arg6,
                      arg7,
                      arg8,
                      arg9,
                      arg10,
                      arg11,
                      arg12,
                      arg13,
                      arg14,
                      arg15
                    );
                    var obj = {};
                    if (typeof callback !== "function") return output;
                    obj[name] = output;
                    obj.error = error;
                    return obj;
                  };
                }
              })(moduleName, key);
            }
          }
        }
      }
    };
    callbackLevel(app.api);
  })();
}
app.handleRequest = async function (request) {
  var url = new URL(request.url);
  if (request.method === "OPTIONS") return app.response("Ok", { status: 200 });
  if (
    app.has(config) &&
    app.has(config.api) &&
    app.has(config.api.authorization)
  ) {
    var authorization = app.has(request.headers)
      ? app.has(request.headers.get)
        ? request.headers.get("authorization")
        : request.headers.authorization
      : "";
    if (!app.has(authorization))
      authorization = url.searchParams.get("authorization");
    if (typeof config.api.authorization === "string") {
      if (authorization !== config.api.authorization)
        return app.response("Unauthorized.", { status: 400 });
    } else {
      if (config.api.authorization.indexOf(authorization) < 0)
        return app.response("Unauthorized.", { status: 400 });
    }
  }
  var parts = url.pathname.split("/");
  parts.shift();
  if (parts.length === 1 && parts[0].trim() === "") parts[0] = "index";
  if (app.indexMode === true) parts = ["index"];
  if (parts.length > 0 && parts[0].trim() !== "") {
    var level = app;
    for (var i = 0; i <= parts.length - 1; i++) {
      var part = app.camelCase(parts[i].trim().split("-").join(" "));
      if (
        typeof level[part] !== "undefined" ||
        typeof level["indexDynamic"] !== "undefined"
      ) {
        level = !(level === app && typeof level["indexDynamic"] !== "undefined")
          ? level[part]
          : level;
        if (i === parts.length - 1) {
          if (
            typeof level === "object" &&
            level !== null &&
            typeof level["index"] === "function"
          ) {
            if (!app.has(request.headers.get("start-only"))) {
              return await level["index"](request);
            } else {
              level["index"](request);
              return app.response("Started.", { status: 200 });
            }
          }
          if (typeof level === "function") {
            if (!app.has(request.headers.get("start-only"))) {
              return await level(request);
            } else {
              level(request);
              return app.response("Started.", { status: 200 });
            }
          }
        }
        if (
          typeof level === "object" &&
          level !== null &&
          typeof level["indexDynamic"] === "function"
        ) {
          if (!app.has(request.headers.get("start-only"))) {
            return await level["indexDynamic"](request);
          } else {
            level["indexDynamic"](request);
            return app.response("Started.", { status: 200 });
          }
        }
      }
    }
  }
  if (typeof localMode === "undefined" || localMode !== true)
    return app.response("Unknown request: " + url.pathname, { status: 400 });
};
if (
  typeof localMode === "undefined" &&
  typeof addEventListener !== "undefined"
) {
  addEventListener("fetch", (event) => {
    event.respondWith(app.handleRequest(event.request));
  });
}

// call in the end
if (typeof app.start === "function") {
  app.start();
}

for (var i = 0; i <= app.startUps.length - 1; i++) {
  if (typeof app.startUps[i] === "function") {
    app.startUps[i]();
  }
}
for (var i = 0; i <= app.startUps.length - 1; i++) {
  if (!(typeof app.startUps[i] === "function")) {
    if (
      typeof localMode !== "undefined" ||
      (typeof appConfig !== "undefined" &&
        app.has(appConfig.frontend) &&
        appConfig.frontend)
    )
      setTimeout(app.startUps[i].callback, app.startUps[i].time);
  }
}
```