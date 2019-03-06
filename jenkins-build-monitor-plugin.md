---
title: jenkins-build-monitor-plugin
date: 2019-03-06 11:32
categories: jenkins
tags: 
- jenkins
- auth
- lua
- openresty
---


``` sh
 docker run --name bm-hsy-proxy -p8000:80 -d openresty/openresty:1.13.6.2-0-alpine-fat
 docker exec -it bm-hsy-proxy bash
     # in the container
        # install dependecy package
        luarocks install lua-resty-http
        vi /etc/nginx/conf.d/default.conf 
```

### in the container: ngx conf /etc/nginx/conf.d/default.conf 
```
    location / {
      set $test 0;
      if ($http_referer !~ ^localhost:8000/view/CM/view/BM_HSY/$ ) {
          set $test  F;
      }
       
      if ( $request_uri !~ ^/view/CM/view/BM_HSY/$ ) {
          set $test  "${test}F";
      }
       
      if ( $request_uri ~* ^/\$stapler/bound.*fetchJobViews$ ) {
          set $test  "${test}T";
      }
       
      if ( $request_uri ~* ^/.*/a0b5bfed/.* ) {
          set $test  "${test}T";
      }
      if ( $request_uri ~* ^/.*build-monitor-plugin/.* ) {
          set $test  "${test}T";
      }
      
      if ($test = FF ) {
          return 403;
      }

      proxy_set_header        Host $host:$server_port;
      proxy_set_header        X-Real-IP $remote_addr;
      proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header        X-Forwarded-Proto $scheme;
       
   
        set $authStr "";
        set $crumbStr "";
        access_by_lua_file "/usr/local/openresty/nginx/conf/jenkins-auth.lua";
        proxy_set_header Authorization $authStr;
        proxy_set_header Jenkins-Crumb $crumbStr;
        # content_by_lua "ngx.say(ngx.var.crumbStr)";
        proxy_pass http://jenkins-server-url;
    } 
```

### in the container: jenkins-auth /usr/local/openresty/nginx/conf/jenkins-auth.lua 
```lua
local http = require "resty.http"
local httpc = http.new()
local authorization = 'Basic '..ngx.encode_base64('username:token')
local crumbUrl='http://jenkins-server-url/crumbIssuer/api/xml?xpath=concat(//crumbRequestField,":",//crumb)'

function mysplit(inputstr, sep)
        if sep == nil then
                sep = "%s"
        end
        local t={}
        for str in string.gmatch(inputstr, "([^"..sep.."]+)") do
                table.insert(t, str)
        end
        return t
end


function getCrumbStr()
   local r, err = httpc:request_uri(crumbUrl, { method = "GET", headers = {
                       ["Authorization"] = authorization
                  } })

    if not r then
        ngx.log(ngx.ERR, err)
        return
    end
    -- read all body
    local body = r.body
    local crumbStr=mysplit(body,':')[2]
    return crumbStr
end

    local crumbStr=getCrumbStr()
    if crumbStr == nil then
         crumbStr=getCrumbStr()
    end
    if crumbStr == nil then
         crumbStr=getCrumbStr()
    end

    ngx.var.authStr=authorization
    ngx.var.crumbStr=crumbStr
  
```
