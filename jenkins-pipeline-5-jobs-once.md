---
title: jenkins pipeline ,every 5 jobs once
date: 2019-03-04 11:32
categories: jenkins
tags: 
- jenkins
- pipeline
---

###  
```
PROJECT_LISTS=[
'project0',
'project1',
'project2',
'project3',
'project4',
'project5',
'project6',
'project7',
'project8',
'project9',
'project10',
'project11',
'project12',
'project13',
'project14',
'project15',
'project16',
'project17',
'project18',
'project19',
]

def transformIntoStep(inputString) {
    return {
        node {
            echo "BUILD__${inputString}"
            build (job: "BUILD__${inputString}",
                        parameters: [[$class: 'StringParameterValue', name: 'BRANCH', value: "${BRANCH}"],],
                        propagate: false)
        }
    }
}

projectGroups=PROJECT_LISTS.collate(5)

node {
    label 'none'
    for (int i = 0; i < projectGroups.size(); i++) {
        def groupNo = i
        stage("group ${groupNo}") {
           def stepsForParallel = projectGroups[i].collectEntries {
                ["echoing ${it}" : transformIntoStep(it)]
            }
            parallel stepsForParallel
        }
    }
}
  
```
