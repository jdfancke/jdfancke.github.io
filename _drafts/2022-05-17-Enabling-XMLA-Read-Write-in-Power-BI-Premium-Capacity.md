---
layout: post
title: 'Enabling XMLA Read Write in Power BI Premium Capacity'
date: 2022-05-13 19:22
summary: 'Enabling XMLA Read Write in Power BI Premium Capacities'
mermaid: true
tag: PBI
---

# Enabling XMLA Read/Write in Power BI Premium Capacity
[JFancke@wabtec.com](mailto:JFancke@wabtec.com)

### Benefits
- ALM Toolkit
	- Diff-based deployments
- Tabular Editor
	- Ability to make changes to the published model without a full publish
- Visual Studio Migration
	- Able to deploy 1500 compatibility level models developed in Visual Studio direct to Power BI Premium.
- Development best practices including source/version control
- Able to avoid full refresh with each development item (measures, individual table additions/edits)
	- Publishing via Power BI does a restore of the .abf and wipes out the existing data model
- Backup and restore of existing .abf files (not confirmed if require xmla endpoint for this)

### Limitations
- A Power BI dataset hosted in the PBI Service edited directly with XMLA currently cannot have it's associated .pbix downloaded from the Power BI service.
	- Note: it is [best practice](https://docs.microsoft.com/en-us/power-bi/guidance/powerbi-implementation-planning-usage-scenario-advanced-data-model-management) to NOT include visuals in the same .pbix as the the .pbix hosting the data model.
	- Note: while it is not possible to download the .pbix directly from the service using the default "download .pbix" button, [there are other ways you can retrieve the model from the service.](https://www.thinkbi.de/2022/05/03/copy-power-bi-data-model-back-to-pbix-file/) Note that incremental refresh disables the ability to download the .pbix as well.
	- Note: the Power BI team is working on [*desktop hardening*](https://ssbipolar.com/2021/12/17/tabular-editor-and-desktop-hardening/#:~:text=Desktop%20hardening%20is%20the%20internal,as%20a%20supported%20public%20API) so that in future it should be possible to download .pbix files in the PBI service that contain data models edited via XMLA

### Other topics:
- Only developers of Power BI objects should be given the **Contributor** or **Member** workspace role. This role enables them to add/update/delete every object in the workspace.
- The **Admin** role should be reserved for users that are responsible for giving others access.
- The **Viewer** role is for all others that need access to see the data but not add/update/delete objects in the workspace.
- Build Access: This gives users the permission equivalent to read-access (in AS) to the dataset. It is managed per dataset. 
	- Contributor, Member, and Admin all receive build access in their role.
	- A user assigned the Viewer role in a workspace can have Build access added per dataset. This allows them to view the data model definitions (e.g. view measure definitions, etc.). Without this they are only able to see the data surfaced via tools that surface specific information (e.g. measures, tables, columns, etc. but not measure *definitions*)

# XMLA Background Information
## What is XMLA?
- [**Markup**](https://en.wikipedia.org/wiki/Markup_language) = A way to give meaning/attributes to plain text
- [**SGML**](https://en.wikipedia.org/wiki/Standard_Generalized_Markup_Language) = Standard General Markup Language (first generalized markup language, 1988)
- [**XML**](https://en.wikipedia.org/wiki/XML) = eXtensible Markup Language (a standard markup language based on the ideas of SGML that can then be used to create other markup languages, 1998)
- [**SOAP**](https://en.wikipedia.org/wiki/SOAP) = Simple Object Access Protocol (an application of XML used for data exchange, 1998)
- [**XMLA**](https://docs.microsoft.com/en-us/analysis-services/multidimensional-models-scripting-language-assl-xmla/developing-with-xmla-in-analysis-services?view=asallproducts-allversions) = eXtensible Markup Language for Analysis services (an application of XML based on SOAP for communicating with Analysis Services, developed by Microsoft/Hyperion/SAS, 2000)

Microsoft Analysis Services uses two protocol **extensions** on top of **XMLA**:
- [**MS-SSAS**](https://docs.microsoft.com/en-us/openspecs/sql_server_protocols/ms-ssas/854a72f2-d637-4be3-b60f-6a44422e80c9) (Microsoft SQL Server Analysis Services): Original protocol for Multidimensional and Datamining models, 2009
	- includes [**ASSL**](https://docs.microsoft.com/en-us/analysis-services/multidimensional-models/scripting-language-assl/developing-with-analysis-services-scripting-language-assl?view=asallproducts-allversions) (Analysis Services Scripting Language): Used for object management of Multidimensional + Tabular compatibility level 1103 and lower
- [**MS-SSAS-T**](https://docs.microsoft.com/en-us/openspecs/sql_server_protocols/ms-ssas-t/f85cd3b9-690c-4bc7-a1f0-a854d7daecd8) (Microsoft SQL Server Analysis Services Tabular): Tabular objects compatibility level 1200+, 2016
	- includes [**TMSL**](https://docs.microsoft.com/en-us/analysis-services/tmsl/tabular-model-scripting-language-tmsl-reference?view=asallproducts-allversions) (Tabular Model Scripting Language): Used for object management of Tabular objects with compatibility level 1200+

### History
<div class="mermaid">
flowchart LR
SGML["SGML<br>1988"] -.-> XML["XML<br>1998"] -.-> SOAP["SOAP<br>1998"] -.-> XMLA -->MS-SSAS("MS-SSAS, 2009<br>MS-SSAS-T, 2016")
subgraph XMLA["XMLA, 2000"]
MS-SSAS
end
</div>

## XMLA Communication
Client applications send XMLA messages over HTTP/HTTPS or TCP to the analysis services instance, then the analysis services instance sends a response back also via XMLA. 

There are 3 types of commands within XMLA: **Authenticate**, **Discover**, and **Execute**.

1. **Authentication** request with provision of credentials upon first contacting the analysis services instance to log into the instance and get a session ID to use in the header for future messages. 
2. **Discover** request asking for metadata information of the data model (e.g. model name, tables, measures, etc.)
3. **Execute** commands when you either want to query data using DAX or MDX, or if you want to update the model metadata and/or process the database using TMSL / ASSL. 

### Example XMLA communication flow
<div class="mermaid">
sequenceDiagram
participant Sender as Power BI Desktop /<br>ALM Toolkit /<br>Tabular Editor /<br>Visual Studio
participant Receiver as Power BI Local Instance /<br>Power BI Workspace /<br>Analysis Services Instance

Sender-->>Receiver: Authenticate<br>Discover<br>Execute<br>(SOAP messages)
activate Receiver
Receiver-->>Sender: AuthenicateResponse<br>DiscoverResponse<br>ExecuteResponse<br>(SOAP messages)
deactivate Receiver
</div>

### Example SOAP Message composition of an Execute -> Command -> Statement command
<div class="mermaid">
flowchart LR

subgraph SOAPENV ["SOAP-ENV: Envelope"]
direction LR
subgraph SOAPENVH["SOAP-ENV: Header"]
SessionInformation["Session Information"]
end
subgraph SOAPENVB["SOAP-ENV: Body"]
subgraph Execute
subgraph Command
subgraph Statement
MDX/DAX/TMSL
end
end
end
end
end
</div>

### XMLA message examples
**Example SOAP message to refresh the AdventureWorks database**
```xml
<Envelope xmlns="http://schemas.xmlsoap.org/soap/envelope/">
  <Header>
		<Session xmlns="urn:schemas-microsoft-com:xml-analysis" SessionId="34B67555-85B9-46CE-8803-4BEC7D6AEE13" />
  </Header>
  <Body>
      <Execute xmlns="urn:schemas-microsoft-com:xml-analysis">
         <Command>
            <Statement>
							{
								"refresh": {
									"type": "automatic",
									"objects": [
										{"database": "AdventureWorks"}
									]
								}
							}
						</Statement>
         </Command>
         <Properties>
					<PropertyList>
					</PropertyList>
         </Properties>
      </Execute>
  </Body>
</Envelope>
```
**Example SOAP message to execute a DAX query against the RMSProduction database**
```xml
<Envelope xmlns="http://schemas.xmlsoap.org/soap/envelope/">
  <Header>
		<Session xmlns="urn:schemas-microsoft-com:xml-analysis" SessionId="34B67555-85B9-46CE-8803-4BEC7D6AEE13" />
  </Header>
  <Body>
      <Execute xmlns = "urn:schemas-microsoft-com:xml-analysis">
				<Command>
					<Statement>EVALUATE Parts</Statement>
				</Command>
				<Properties>
					<PropertyList>
						<Catalog>RMSProduction</Catalog>
					</PropertyList>
				</Properties>
			</Execute>
  </Body>
</Envelope>
```

## Client Libraries
Client applications do not communicate with the Analysis Services instance using raw XMLA inputs/outputs - instead they use client libraries which allow programmatic communication to the Analysis Services instance via XMLA (converting commands in the client libraries into the appropriate XMLA message). 
Essentially these are the libraries that client applications use to more easily build solutions that communicate with the analysis services server (as opposed to only using the open standards of XMLA and the MS-SSAS / MS-SSAS-T protocols).

Virtually all client applications (including Power BI Desktop, ALM Toolkit (developed by the Power BI Senior Program Manager at Microsoft), Tabular Editor, and DAX Studio) use these same client libraries.

Libraries:
- ADOMD
- AMO
- MSOLAP



