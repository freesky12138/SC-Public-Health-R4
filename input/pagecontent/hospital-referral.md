## 业务背景

&emsp;&emsp;“双向转诊”，简而言之就是“小病进社区，大病进医院”，积极发挥大中型医院在人才、技术及设备等方面的优势，同时充分利用各社区医院的服务功能和网点资源，促使基本医疗逐步下沉社区，社区群众危重病、疑难病的救治到大中型医院。

## 场景简介

&emsp;&emsp;医生根据患者个人意愿并综合患者病情、转入医院学科特长和床位占用情况等信息发送转诊预约申请请求，经转入医院医生审核确认同意后反馈转出医院，告知患者，同步传送电子病历、检查检验结果信息和（或）电子健康档案信息，患者到诊后，回传患者到诊消息通知。主要业务过程包括床位信息查询、转诊预约申请提交、申请审核与确认、审核应答、履约情况确认、病历上传等。

## 场景流程

### 定义流程的资源

&emsp;&emsp;场景所涉及的流程及流程中的活动是通过以下两个FHIR的资源来定义的：

- 活动定义：[ActivityDefinition](http://www.hl7fhir.cn/R4/activitydefinition.html)：活动定义资源，定义在业务流程中每一个活动步骤，描述其在流程中的作用。
- 流程定义：[PlanDefinition](http://www.hl7fhir.cn/R4/plandefinition.html)：通过计划义定资源可以对活动定义资源进行组装，并描述活动之间的先后关系，触发条件等信息，该资源可描述一个完整的业务流程。


### 场景中的活动与流程的定义
  
 双向转诊流-住院流程分为“预约申请”、“预约应答”、“到诊应答”、“上传病历”四个步骤，
使用[ActivityDefinition](http://www.hl7fhir.cn/R4/activitydefinition.html)资源分别定义如下：

- [转诊预约申请-ActivityDefinition定义](ActivityDefinition-ActivityDefinition-application-for-referral-appointment.html)
- [转诊预约应答-ActivityDefinition定义](ActivityDefinition-ActivityDefinition-application-for-referral-appointment-response.html)
- [患者到诊应答-ActivityDefinition定义](ActivityDefinition-ActivityDefinition-patient-arrive-response.html)
- [上传完整病历-ActivityDefinition定义](ActivityDefinition-ActivityDefinition-medical-records-submitted.html)

> 使用[PlanDefinition](http://www.hl7fhir.cn/R4/plandefinition.html)资源定义双向转诊-住院的业务流程：[住院双转流程-PlanDefinition定义](PlanDefinition-PlanDefinition-hospital-referral.html)，
它通过其Action元素将以上四个步骤组装起来。每个Action关联一个步骤。可以通过此流程定义资源实例实现流程自动化。
转诊预约申请活动为该流程的第一步，转诊申请发起后，根据转诊预约应答来确定后续步骤，如果为不同意，结束流程，如果为同意，等待患者到诊应答，患者到诊应答结束后，上传完整病历信息。

### 活动实例资源
 [ActivityDefinition](http://www.hl7fhir.cn/R4/activitydefinition.html)活动定义中有一个kind元素，此元素描述了该活动具体使用哪种资源类型,即活动实例资源类型。
比如[转诊预约申请-ActivityDefinition定义](ActivityDefinition-ActivityDefinition-application-for-referral-appointment.html)的活动实例资源类型为继承自Appointment资源类型的[HospitalReferral](StructureDefinition-hospital-referral.html)资源。
此场景中涉及的活动实例资源如下：

- [HospitalReferral](StructureDefinition-hospital-referral.html):转诊预约申请资源，该资源描述医院转诊的的申请。包括上转、下转都使用该资源。
- [HospitalReferralResponse](StructureDefinition-hospital-referral-response.html)：转诊预约应答资源，该资源描述在提交转诊申请后，由接收方给出是否同意的转诊应答。
- [Task](http://www.hl7fhir.cn/R4/task.html):任务资源，再该场景下描述上传病历的步骤任务。

上述流程定义、活动定义、活动实例资源的关系图见下：
![流程定义](PlanDefinition-ActivityDefinition-Task-Relationship.png)


> 流程启动后，每个活动步骤将产生一个活动实例资源的新实例，其示例如下：

- [转诊预约申请示例](Appointment-HospitalReferral-example.html)
- [转诊预约应答示例](AppointmentResponse-HospitalReferralResponse-example.html)
- [患者到诊应答示例](AppointmentResponse-PatientArriveResponse-example.html)
- [上传完整病历示例](Task-Medical-records-submitted-example.html)



## 数据交换方式与结构

FHIR的数据交换方式支持RESTful、SOA、消息交换等。本场景目前只定义了消息交换的方式，其他方式在进一步定义中。

[FHIR消息交换框架](http://www.hl7fhir.cn/R4/messaging.html)要求的应用程序符合“ FHIR消息传递”，它支持大多数消息组件、ESB等消息引擎，支持异步消息和消息应答机制。

数据交换使用的消息体（报文）通过[Bundle](http://www.hl7fhir.cn/R4/bundle.html)（资源捆束）组装数据。
将[Bundle](http://www.hl7fhir.cn/R4/bundle.html)资源作用消息传输时，其[Bundle.type](http://www.hl7fhir.cn/R4/bundle-definitions.html#Bundle.type)元素固定为“message”，
且资源捆束内的第一个条目(entry)中的资源必须是[MessageHeader](http://www.hl7fhir.cn/R4/messageheader.html)（消息头）。其它条目中包含上述活动实例资源和相关业务资源。
资源捆束里的各条目中的资源的相互关系是通过消息头引用活动实例资源，活动实例资源再关联业务资源来展现的。示意图如下：

![数据结构](structure-bundle.png)

### 消息体结构定义
资源捆束中条目中的资源类型在不同的业务活动（步骤）中是不同的，
本规范集使用[MessageDefinition](http://www.hl7fhir.cn/R4/messagedefinition.html)定义每个活动数据交换的消息体结构，
遵循[住院双转流程定义](PlanDefinition-PlanDefinition-hospital-referral.html)中的Action元素对应的步骤定义。为每个步骤分别定义了消息体的具体结构，如下示例：

- [转诊预约申请-MessageDefinition定义](MessageDefinition-MessageDefinition-hospital-referral.html)：该消息定义为 双向转诊-住院场景中定义的消息，在转出医疗机构向转入医疗结构发起转诊预约申请时使用，该消息定义描述了在整个消息传输过程中必须具备的资源结构，包括 第一个资源 必须为MessageHeader资源（1..1）,第二个资源必须为Appointment资源(1..1),可以包含其他Resource资源（0..*），并且该消息具备2个应答消息：1.转入医院收到转诊申请后，根据自己医院情况和患者病情，审核是否通过转诊，审批通过后，作出应答。2.患者到达转入医院，办理手续后，发起患者到诊应答。
- [转诊预约应答-MessageDefinition定义](MessageDefinition-MessageDefinition-hospital-referral-response.html)：该消息定义为 双向转诊-住院场景中定义的消息，在转入医疗机构收到转入医疗机构发起的转诊申请，并且审批是否通过后使用，该消息定义描述了在整个消息传输过程中必须具备的资源结构，包括 第一个资源 必须为MessageHeader资源（1..1）,第二个资源必须为AppointmentResponse资源(1..1),可以包含其他Resource资源（0..*），并且该消息不需要应答。
- [患者到诊应答-MessageDefinition定义](MessageDefinition-MessageDefinition-patient-arrive-response.html)：该消息定义为 双向转诊-住院场景中定义的消息，在转出医院接收到转诊预约应答，并且通过审批后，患者到达转入医疗机构，发送患者到诊应答消息，使用该消息发送患者到诊应答信息，该消息定义描述了在整个消息传输过程中必须具备的资源结构，包括 第一个资源 必须为MessageHeader资源（1..1）,第二个资源必须为AppointmentResponse资源(1..1),可以包含其他Resource资源（0..*），并且该消息具备1个应答消息：患者到达转入医院，办理手续后，收到该患者到诊应答的消息，通过该消息上传完整病历作为应答。
- [上传完整病历-MessageDefinition定义](MessageDefinition-MessageDefinition-medical-records-submitted.html)：该消息定义为 双向转诊-住院场景中定义的消息，在转出医疗机构收到患者到诊应答消息后，使用该消息上传完整病历信息，该消息定义描述了在整个消息传输过程中必须具备的资源结构，包括 第一个资源 必须为MessageHeader资源（1..1）,第二个资源必须为Task资源(1..1),可以包含其他Resource资源（0..*），并且该消息不需要应答。

### 流程资源和业务资源关系  

> 介绍流程资源和业务资源的相互关联关系，关系图如下：
  
![业务类图](Class.png)

- [MedicalRecordDocumentation](https://build.fhir.org/ig/HL7China/CN-CORE-R4/branches/develop/StructureDefinition-medical-record-documentation.html)：病历引用资源，引用第三方的病历文书，并且把病历文书作为附件形式上传。
- [HospitalBed](https://build.fhir.org/ig/HL7China/CN-CORE-R4/branches/develop/StructureDefinition-hospital-bed.html)：病床信息资源，描述医院床位的基础信息以及当前状态。
- [Patient](https://build.fhir.org/ig/HL7China/CN-CORE-R4/branches/develop/StructureDefinition-Patient.html)：患者资源，接受医疗健康服务的个人或动物，医疗过程是以患者为中心的。对交叉索进行中国本地化约定。
- [Practitioner](https://build.fhir.org/ig/HL7China/CN-CORE-R4/branches/develop/StructureDefinition-Practitioner.html)：医护人员资源，直接或间接参与提供医疗健康服务的人员。
- [Department](https://build.fhir.org/ig/HL7China/CN-CORE-R4/branches/develop/StructureDefinition-Department.html)：科室/部门资源，描述医院科室/部门的基础信息。
- [Hospital](https://build.fhir.org/ig/HL7China/CN-CORE-R4/branches/develop/StructureDefinition-Hospital.html)：医院资源，描述医疗机构（医院）的基本信息。
- [PractitionerRole](https://build.fhir.org/ig/HL7China/CN-CORE-R4/branches/develop/StructureDefinition-PractitionerRole.html)：医护人员工作信息资源，医务人员提供医疗服务时的岗位相关信息，包括所属组织、科室、角色/岗位等。
- [List](http://www.hl7fhir.cn/R4/list.html)：集合资源，在该场景下对患者的完整病历进行打包操作。

## 数据交互流程

数据交互流程遵循[PlanDefinition](http://www.hl7fhir.cn/R4/plandefinition.html)定义的流程规则和步骤，按照[MessageDefinition](http://www.hl7fhir.cn/R4/messagedefinition.html)定义每个消息体结构组装数据

双向转诊-住院 数据交互主要包括两类：

- 转入医院和转出医院都具备自己的系统，并且和双向转诊平台做数据接入，实现功能。
- 转入医院不需要和双向转诊信息平台做系统对接，所有业务在双向转诊信息平台上直接操作。
  
### 通过双转平台对接

![流程图](sequence-platform.png)
  
> 具体流程：

1. 转出医院根据获取到的转入医院的科室床位资源情况，发起转诊预约申请，附带基本病情介绍，发送转诊预约申请经过发送到 双转平台。该流程符合流程定义[住院双转流程定义](PlanDefinition-PlanDefinition-hospital-referral.html)中的第一个action节点定义的步骤-转诊预约申请，按照[转诊预约申请消息定义](MessageDefinition-MessageDefinition-hospital-referral.html)的消息结构定义使用[Bundle](http://www.hl7fhir.cn/R4/bundle.html)组装数据并发送，具体示例参见： [转诊预约申请消息交换示例](Bundle-Bundle-hospital-referral-example.html)。
2. 双转平台根据实际情况进行审批，通过双转平台下发转诊预约应答到转出医院。该流程符合流程定义[住院双转流程定义](PlanDefinition-PlanDefinition-hospital-referral.html)中的第二个action节点定义的步骤-转诊预约应答，按照[转诊预约应答消息定义](MessageDefinition-MessageDefinition-hospital-referral-response.html)的消息结构定义使用[Bundle](http://www.hl7fhir.cn/R4/bundle.html)组装数据并发送，具体示例参见：[转诊预约应答消息交换示例](Bundle-Bundle-hospital-referral-response-example.html)。
3. 患者到转入医院报到后，办理入院手续，双转平台回传患者到诊应答到转出医院。该流程符合流程定义[住院双转流程定义](PlanDefinition-PlanDefinition-hospital-referral.html)中的第三个action节点定义的步骤-患者到诊应答，按照[患者到诊应答消息定义](MessageDefinition-MessageDefinition-patient-arrive-response.html)的消息结构定义使用[Bundle](http://www.hl7fhir.cn/R4/bundle.html)组装数据并发送，具体示例参见：[患者到诊应答消息交换示例](Bundle-Bundle-patient-arrive-response-example.html)。
4. 根据到诊通知，转出医院上传该患者相关的病历资料到双转平台。该流程符合流程定义[住院双转流程定义](PlanDefinition-PlanDefinition-hospital-referral.html)中的第四个action节点定义的步骤-上传完整病历，按照[上传完整病历消息定义](MessageDefinition-MessageDefinition-medical-records-submitted.html)的消息结构定义使用[Bundle](http://www.hl7fhir.cn/R4/bundle.html)组装数据并发送，具体示例参见：[上传完整病历消息交换示例](Bundle-Bundle-medical-records-submitted-example.html)。

### 通过双转平台操作

![流程图](sequence.png)
  
> 具体流程：

1. 转出医院根据获取到的双转平台的科室床位资源情况，发起转诊预约申请，附带基本病情介绍发送到双转平台，该流程符合流程定义[住院双转流程定义](PlanDefinition-PlanDefinition-hospital-referral.html)中的第一个action节点定义的步骤-转诊预约申请，按照[转诊预约申请消息定义](MessageDefinition-MessageDefinition-hospital-referral.html)的消息结构定义使用[Bundle](http://www.hl7fhir.cn/R4/bundle.html)组装数据并发送，具体示例参见： [转诊预约申请消息交换示例](Bundle-Bundle-hospital-referral-example.html)。
2. 双转平台根据实际情况进行审批，下发转诊预约应答到转出医院。该流程符合流程定义[住院双转流程定义](PlanDefinition-PlanDefinition-hospital-referral.html)中的第二个action节点定义的步骤-转诊预约应答，按照[转诊预约应答消息定义](MessageDefinition-MessageDefinition-hospital-referral-response.html)的消息结构定义使用[Bundle](http://www.hl7fhir.cn/R4/bundle.html)组装数据并发送，具体示例参见：[转诊预约应答消息交换示例](Bundle-Bundle-hospital-referral-response-example.html)。
3. 患者到转入医院报到后，办理入院手续，双转平台回传患者到诊通知到转出医院。该流程符合流程定义[住院双转流程定义](PlanDefinition-PlanDefinition-hospital-referral.html)中的第三个action节点定义的步骤-患者到诊应答，按照[患者到诊应答消息定义](MessageDefinition-MessageDefinition-patient-arrive-response.html)的消息结构定义使用[Bundle](http://www.hl7fhir.cn/R4/bundle.html)组装数据并发送，具体示例参见：[患者到诊应答消息交换示例](Bundle-Bundle-patient-arrive-response-example.html)。
4. 根据到诊通知，转出医院上传该患者相关的病历资料到双转平台，该流程符合流程定义[住院双转流程定义](PlanDefinition-PlanDefinition-hospital-referral.html)中的第四个action节点定义的步骤-上传完整病历，按照[上传完整病历消息定义](MessageDefinition-MessageDefinition-medical-records-submitted.html)的消息结构定义使用[Bundle](http://www.hl7fhir.cn/R4/bundle.html)组装数据并发送，具体示例参见：[上传完整病历消息交换示例](Bundle-Bundle-medical-records-submitted-example.html)。
