---
title: sonarqube update project visibility to private
date: 2020-03-26 15:32
categories: sonarqube
tags: 
- sonarqube
---


### demo-code
``` python2
def set_project_private():
    payload = {
        'ps': '500'

    }
    requests.packages.urllib3.disable_warnings()
    r = requests.get("https://'+SONAR_SERVER_URL+'/api/projects/search",
                     auth=HTTPBasicAuth(SONAR_ADMIN, SONAR_PASSWORD),
                     params=payload,
                     verify=False)

    json_data = json.loads(r.text)
    for p in json_data['components']:
        set_private(p['key'])


def set_private(p_key):
    payload = {
        'project': p_key,
        'visibility': 'private'
    }
    requests.packages.urllib3.disable_warnings()

    r = requests.post("https://'+SONAR_SERVER_URL+'/api/projects/update_visibility",
                      auth=HTTPBasicAuth(SONAR_ADMIN, SONAR_PASSWORD),
                      data=payload,
                      verify=False)
    print r
    print r.text
    # json_data = json.loads(r.text)

SONAR_'+SONAR_SERVER_URL+'=
SONAR_ADMIN='admin'
SONAR_PASSWORD=

if __name__ == "__main__":
    set_project_private()

```
