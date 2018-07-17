---
title:  service health check , generate error dependency graph
date: 2018-07-17 13:53
categories: graphviz
tags: 
- edas
- python
- graphviz
---

# 解析所有服务的health.html的返回,针对错误的节点生成图
``` python
#!/usr/bin/python2
#-*- coding: UTF-8 -*-
import sys
import requests
import json
from graphviz import Digraph



reload(sys)
sys.setdefaultencoding('utf-8')

class Service:
    def __init__(self, cid,group, id, qualifiedInterface,alive):
        self.cid=cid
        self.group = group
        self.id = id
        self.qualifiedInterface = qualifiedInterface
        self.alive = alive

    def __repr__(self):
        return str(self.__dict__)


class Node:
   def __init__(self, name, cid,aliveAndWell):
      self.name = name
      self.cid = cid
      self.aliveAndWell=aliveAndWell
      self.consumers = []
      self.providers = []

   def __repr__(self):
       return str(self.__dict__)

def health_check(name, cid):
    port='3'+ cid.replace('C',"")
    #health_json_str = requests.get("http://172.18.23.234:%s/health.html"%(port) ).text
    health_json_str = requests.get("http://172.18.4.22:%s/health.html"%(port) ).text
    health_json = json.loads(health_json_str)
    node=Node(name, cid,health_json['pong']['aliveAndWell'])
    #print health_json['pong']['aliveAndWell']
    for c in health_json['pong']['consumers']:
        c_service=Service(cid,c['group'], c['id'], c['qualifiedInterface'],c['alive'])
        node.consumers.append(c_service)

    for p in health_json['pong']['consumers']:
        p_service=Service(cid,p['group'], p['id'], p['qualifiedInterface'],p['alive'])
        node.providers.append(p_service)

    return node

def main():
    try:
        nodes=[]
        f = open('hsy_list.txt', 'r')
        #print f.read()
        lines=f.read().split('\n')
        for line in lines:
            if len(line.strip()) > 0:
                name = line.strip().split(' ')[0]
                cid     = line.strip().split(' ')[1]
                node=health_check(name,cid)
                nodes.append(node)


        dot = Digraph(comment='The Round Table')

        providers_dict={}
        for node in nodes:
            if node.aliveAndWell:
                continue
            dot.node(node.cid,node.name)
            for p in node.providers:
                providers_dict[p.qualifiedInterface]=node

        for node in nodes:
            if node.aliveAndWell:
                continue
            for c in node.consumers:
                if not c.alive and  providers_dict.has_key(c.qualifiedInterface):
                    dot.edge(providers_dict[c.qualifiedInterface].cid,node.cid,label=c.qualifiedInterface)

        print(dot.source)
        dot.render('round-table.gv', view=True )


    finally:
        if f:
            f.close()



if __name__ == '__main__':
    main()
```

# hsy_list.txt
```
paas-generic-identifier C0879
paas-generic-metadata C7395
paas-generic-party C5381
paas-generic-graphql C3075
paas-intg-third-srv C8753
paas-generic-config C6485
paas-generic-product C1649
paas-support-inventory C1024
paas-support-arap C5430
paas-generic-uom C8973
paas-generic-bizconcept C7813
paas-generic-authorization C3240
paas-generic-pushcomm C2315
```
# http://172.18.23.234:%s/health.html  返回json
```
{"pong":{"aliveAndWell":true,"appsvrStatus":{"alive":true,"attributes":{"localAddress":"127.0.0.1","localPort":35381},"endpoint":{"address":"172.18.23.234","port":35381},"state":"ALIVE","type":"AppSvr"},"consumers":[{"alive":true,"buildVersion":{"deployTime":"20180716185022","methodSignatures":["bindDataAuth(String,String)IResult","findUserDataAuthList(Long)IListResult","saveUserDataAuth(Long,Map)IResult","findDataAuthSpecList()IListResult","onlyEditOwnVoucher(String)IResult","obtainServiceVersion()IResult","getServiceContext()IServiceContext"],"serviceAddress":"127.0.0.1","serviceVersion":"develop-1.0.0"},"group":"paasGenericGroup","id":"dataAuthService","processState":"ALIVE","qualifiedInterface":"com.chanjet.paas.generic.authorization.service.DataAuthService","ref":"","version":"1.0.0"},{"alive":true,"buildVersion":{"deployTime":"20180716185022","methodSignatures":["setUserAuthType(Long,UserAuthTypeEnum)IResult","clearUserAuthData(Long)IResult","findRoleAuthResourceTransactions(List,String,AuthClassEnum)IListResult","findRoleAuthResourceTransactions(List,Long,AuthClassEnum)IListResult","findUserAllAuthorizedResourceTransactions()IListResult","findUserAuthResourceTransactions(List,Long,AuthClassEnum)IListResult","saveRoleAuth(Long,List,List,AuthClassEnum)IResult","saveUserAuth(Long,List,List,AuthClassEnum)IResult","filterRolePermissions(Map)IResult","copyUserAuth(Long,List,List)IResult","notifyAuthCacheExpired(Long)IResult","obtainServiceVersion()IResult","getServiceContext()IServiceContext"],"serviceAddress":"127.0.0.1","serviceVersion":"develop-1.0.0"},"group":"paasGenericGroup","id":"userAuthService","processState":"ALIVE","qualifiedInterface":"com.chanjet.paas.generic.authorization.service.UserAuthService","ref":"","version":"1.0.0"},{"alive":true,"buildVersion":{"deployTime":"20180716185156","methodSignatures":["batchRemoveDeptTest(List)IResult","initData()IResult","findByParentId(Long,long)List","addDepartment(Department)IResult","updateDepartment(Department)IResult","removeDepartment(Long)IResult","findDepartment(Long)IResult","findAllActiveDepartment()IListResult","isHeadquarters(Long)IResult","repairDataOverridden()IResult","repairData()IResult","obtainServiceVersion()IResult","getServiceContext()IServiceContext"],"serviceAddress":"127.0.0.1","serviceVersion":"develop-1.0.0"},"group":"paasGenericGroup","id":"departmentService","processState":"ALIVE","qualifiedInterface":"com.chanjet.paas.generic.tenant.service.DepartmentService","ref":"","version":"1.0.0"},{"alive":true,"buildVersion":{"deployTime":"20180716185156","methodSignatures":["addEmployee(Employee)IResult","updateEmployee(Employee)IResult","removeEmployee(Long)IResult","authEmployee(Long,Long)IResult","unAuthEmployee(Long,Long)IResult","findUserLincese()IResult","findEmployee(Long)IResult","findAllActiveEmployee()IListResult","syncCIAtest(String)IResult","batchRemoveEmployeeTest(List)IResult","obtainServiceVersion()IResult","getServiceContext()IServiceContext"],"serviceAddress":"127.0.0.1","serviceVersion":"develop-1.0.0"},"group":"paasGenericGroup","id":"employeeService","processState":"ALIVE","qualifiedInterface":"com.chanjet.paas.generic.tenant.service.EmployeeService","ref":"","version":"1.0.0"},{"alive":true,"buildVersion":{"deployTime":"20180716185156","methodSignatures":["findOne(Long)IResult","findByIds(List)IListResult","authUser(String,String)IResult","isAdmin(Long)IResult","obtainServiceVersion()IResult","getServiceContext()IServiceContext"],"serviceAddress":"127.0.0.1","serviceVersion":"develop-1.0.0"},"group":"paasGenericGroup","id":"userService","processState":"ALIVE","qualifiedInterface":"com.chanjet.paas.generic.tenant.service.UserService","ref":"","version":"1.0.0"},{"alive":true,"buildVersion":{"deployTime":"20180716184816","methodSignatures":["returnCode(CodeReturnContext)IResult","generatePrepareCodes(String,JSONObject)IResult","generateCodes(String,JSONObject)IResult","prepareGenerate(CodeContext)IResult","updateMaxCode(CodeContext,Integer)IResult","generateIds(String,Integer)IResult","generateId(String)IResult","generate(CodeContext)IResult","obtainServiceVersion()IResult","getServiceContext()IServiceContext"],"serviceAddress":"127.0.0.1","serviceVersion":"develop-1.0.0"},"group":"paasGenericGroup","id":"identifierService","processState":"ALIVE","qualifiedInterface":"com.chanjet.paas.generic.identifier.service.IdentifierService","ref":"","version":"1.0.0"},{"alive":true,"buildVersion":{"deployTime":"20180716185021","methodSignatures":["batchAddBoLabelDTO(BoLabelsDTO)IResult","batchAddBoLabelDTORemoveOld(BoLabelsDTO)IResult","addBoLabels(BoLabels)IResult","batchAddBoLabels(List)IResult","removeBoLabels(Long)IResult","batchRemoveBoLabels(List)IResult","removeBoLabelsByBoNameAndBoId(Long,String)IResult","removeBoLabelsByBoNameAndBoIds(List,String)IResult","batchAddSysLabels(String,List,String)IResult","batchRemoveSysLabels(String,String)IResult","checkExists(Long,String,Long)Boolean","obtainServiceVersion()IResult","getServiceContext()IServiceContext"],"serviceAddress":"127.0.0.1","serviceVersion":"develop-1.0.0"},"group":"paasGenericGroup","id":"boLabelsService","processState":"ALIVE","qualifiedInterface":"com.chanjet.paas.generic.accs.service.BoLabelsService","ref":"","version":"1.0.0"},{"alive":true,"buildVersion":{"deployTime":"20180716184816","methodSignatures":["register(String)Result","obtainServiceVersion()IResult","getServiceContext()IServiceContext"],"serviceAddress":"127.0.0.1","serviceVersion":"develop-1.0.0"},"group":"paasGenericGroup","id":"serverRegisterService","processState":"ALIVE","qualifiedInterface":"com.chanjet.paas.generic.identifier.service.ServerRegisterService","ref":"","version":"1.0.0"}],"dbmsStatus":{"alive":true,"attributes":{"username":"hsy@172.18.23.234"},"endpoint":{"address":"172.16.33.45","port":3306},"name":"MySQL","state":"ALIVE","type":"DBMS","version":"5.7.18-log"},"providers":[{"alive":true,"buildVersion":{"deployTime":"20180716184817","methodSignatures":["findOne(Long)IResult","obtainServiceVersion()IResult","getServiceContext()IServiceContext"],"serviceAddress":"127.0.0.1","serviceVersion":"develop-1.0.0"},"group":"paasGenericGroup","id":"partyTypeServiceProvider","processState":"ALIVE","qualifiedInterface":"com.chanjet.paas.generic.party.service.PartyTypeService","ref":"partyTypeService","version":"1.0.0"},{"alive":true,"buildVersion":{"deployTime":"20180716184817","methodSignatures":["findAll(List)IListResult","batchUpdateCustVendorField(String,Object,Boolean,List)IResult","initData()IResult","addCustVendor(CustVendor)IResult","addCustVendorForHkj(CustVendor)IResult","updateCustVendor(CustVendor,Boolean)IResult","updateCustVendorForHkj(CustVendor,Boolean)IResult","findPartyRolesByPartyIdAndRoleType(Long,PartyRoleTypeEnum)List","removeCustVendor(Long)IResult","removeCustVendorForHkj(Long,Long)IResult","batchRemoveCustVendor(List)IResult","findCustVendor(Long)IResult","importCustVendor(CustVendor,String)IResult","batchImportCustVendor(List,String)IResult","addCustVendorForShopUser(Long,Long,Long,String,String,String)IResult","calcDueDate(Long,Date)IResult","updateCustVendorForBillingPeriod(Long,String,Integer,Integer,Integer,Integer,Date,Integer)IResult","repairData()IResult","repairDataOverridden()IResult","obtainServiceVersion()IResult","getServiceContext()IServiceContext"],"serviceAddress":"127.0.0.1","serviceVersion":"develop-1.0.0"},"group":"paasGenericGroup","id":"custVendorServiceProvider","processState":"ALIVE","qualifiedInterface":"com.chanjet.paas.generic.party.service.CustVendorService","ref":"custVendorService","version":"1.0.0"},{"alive":true,"buildVersion":{"deployTime":"20180716184817","methodSignatures":["updateCustVendorContact(CustVendorContact)IResult","removeCustVendorContact(Long)IResult","addCustVendorContact(CustVendorContact)IResult","obtainServiceVersion()IResult","getServiceContext()IServiceContext"],"serviceAddress":"127.0.0.1","serviceVersion":"develop-1.0.0"},"group":"paasGenericGroup","id":"custVendorContactServiceProvider","processState":"ALIVE","qualifiedInterface":"com.chanjet.paas.generic.party.service.CustVendorContactService","ref":"custVendorContactService","version":"1.0.0"},{"alive":true,"buildVersion":{"deployTime":"20180716184817","methodSignatures":["findOne(Long)IResult","findAll(ICondition,Sort)Iterable","findByParentId(Long)Iterable","obtainServiceVersion()IResult","getServiceContext()IServiceContext"],"serviceAddress":"127.0.0.1","serviceVersion":"develop-1.0.0"},"group":"paasGenericGroup","id":"partyRoleTypeServiceProvider","processState":"ALIVE","qualifiedInterface":"com.chanjet.paas.generic.party.service.PartyRoleTypeService","ref":"partyRoleTypeService","version":"1.0.0"},{"alive":true,"buildVersion":{"deployTime":"20180716184817","methodSignatures":["batchImportCustVendorCategory(List,Map)IResult","updatePartyCategory(PartyCategory)IResult","removePartyCategory(Long)IResult","findUnclassified()IResult","addPartyCategory(PartyCategory)IResult","obtainServiceVersion()IResult","getServiceContext()IServiceContext"],"serviceAddress":"127.0.0.1","serviceVersion":"develop-1.0.0"},"group":"paasGenericGroup","id":"partyCategoryServiceProvider","processState":"ALIVE","qualifiedInterface":"com.chanjet.paas.generic.party.service.PartyCategoryService","ref":"partyCategoryService","version":"1.0.0"},{"alive":true,"buildVersion":{"deployTime":"20180716184817","methodSignatures":["addMshopUser(MshopUser)IResult","untieMshopUserWithCustVendorContact(Long,Long)IResult","removeMshopUser(Long)IResult","disableMshopUser(Long)IResult","enableMshopUser(Long)IResult","transferMshopUserAdmin(Long)IResult","refreshLastLoginStamp(Long)IResult","obtainServiceVersion()IResult","getServiceContext()IServiceContext"],"serviceAddress":"127.0.0.1","serviceVersion":"develop-1.0.0"},"group":"paasCoreGroup","id":"mshopUserServiceProvider","processState":"ALIVE","qualifiedInterface":"com.chanjet.paas.generic.party.service.MshopUserService","ref":"mshopUserService","version":"1.0.0"},{"alive":true,"buildVersion":{"deployTime":"20180716184817","methodSignatures":["addPartyRoleAttribute(PartyRoleAttribute)IResult","obtainServiceVersion()IResult","getServiceContext()IServiceContext"],"serviceAddress":"127.0.0.1","serviceVersion":"develop-1.0.0"},"group":"paasGenericGroup","id":"partyRoleAttributeServiceProvider","processState":"ALIVE","qualifiedInterface":"com.chanjet.paas.generic.party.service.PartyRoleAttributeService","ref":"partyRoleAttributeService","version":"1.0.0"},{"alive":true,"buildVersion":{"deployTime":"20180716184817","methodSignatures":["export()IResult","obtainServiceVersion()IResult","getServiceContext()IServiceContext"],"serviceAddress":"127.0.0.1","serviceVersion":"develop-1.0.0"},"group":"paasGenericGroup","id":"partyDataExportServiceProvider","processState":"ALIVE","qualifiedInterface":"com.chanjet.paas.generic.party.service.PartyDataExportService","ref":"partyDataExportService","version":"1.0.0"},{"alive":true,"buildVersion":{"deployTime":"20180716184817","methodSignatures":["importData(String,Map,Map)IResult","obtainServiceVersion()IResult","getServiceContext()IServiceContext"],"serviceAddress":"127.0.0.1","serviceVersion":"develop-1.0.0"},"group":"paasGenericGroup","id":"partyDataImportServiceProvider","processState":"ALIVE","qualifiedInterface":"com.chanjet.paas.generic.party.service.PartyDataImportService","ref":"partyDataImportService","version":"1.0.0"}],"redisStatus":{"alive":true,"attributes":{"ping":true},"endpoint":{"address":"172.18.18.208","port":8123},"name":"Redis","state":"ALIVE","type":"Redis"}},"health":"well"}

```
