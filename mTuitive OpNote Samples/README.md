#mTuitive OpNote CCDA Samples

These samples are intended to comply with the [Consolidated CDA Implementation Guide (IG)](http://www.hl7.org/implement/standards/product_brief.cfm?product_id=258).

[mTuitive OpNote](http://mtuitive.com/opnote) is a web-based application that lets surgeons easily create
structured operative reports instead of narrative dictation.  This data is useful for research, clinical automation,
billing, etc.

CCDA is intended to provide some guidance about structured various clinical documents and is already very useful
for exchanging operative reports. The choice of open CDA templates proved very helpful, allowing us to include additional
sections not covered by the IG and entries not specified by the IG.

##Areas for Improvement
The current implementation by mTuitive is useful as-is, but there are several areas that could use additional refinement:
* Encode diagnoses as ICD10 or SNOMED.  Since ICD9 is used in production, that's what we've provided here.
* More entry level section templates
* H&P, Discharge Summary, and Procedure Note implementations
* LOINC codes for all section templates

##Suggestions for CCDA 2.0

The difference between operative note and procedure note is murky. We would be in favor of consolidating them into
one template or providing applicable CPT codes for each.

Real-world operative reports often document multiple procedures. The specification clearly only allows one instance of 
some important required sections, which makes this difficult. In mTuitive's production application, each procedure has
its own instance of some fields. 

To implement this with CCDA, we include a serviceEvent for each procedure. Within the document body, we use a container 
section that corresponds to each serviceEvent being documented. Each container section includes a ProcedureContext 
template to set the context for sub-sections within its scope to the appropriate procedure.  This works but is not 
strictly compliant because there may be multiple operative description or laterality fields in the document. 
We felt this was preferable to concatenating all of the relevant fields into one text block.

For example:

    <section>
          <title>Operative Description</title>
          <component>
            <section>
              <title>Extracapsular cataract removal with insertion of intraocular lens prosthesis (1 stage procedure), manual or mechanical technique (eg, irrigation and aspiration or phacoemulsification) (66984)</title>
              <entry>
                <!-- We want to scope this section to the procedure that its subsections are documenting. -->
                <act moodCode="EVN" classCode="ACT">
                  <templateId root="2.16.840.1.113883.10.20.6.2.5" />
                  <!-- Procedure Context template to let us know exactly which procedure the sections in this scope document.-->
                  <code code="66984" displayName="Extracapsular cataract removal with insertion of intraocular lens prosthesis (1 stage procedure), manual or mechanical technique (eg, irrigation and aspiration or phacoemulsification)" codeSystem="2.16.840.1.113883.6.12" codeSystemName="CPT4" />
                </act>
              </entry>
              <component>
                <section>
                  <title>Laterality</title>
                  <text>
                    <list>
                      <item>Bilateral</item>
                    </list>
                  </text>
                </section>
              </component>
              <component>
                <section>
                  <templateId root="2.16.840.1.113883.10.20.22.2.27" />
                  <code code="29554-3" codeSystem="2.16.840.1.113883.6.1" codeSystemName="LOINC" displayName="PROCEDURE DESCRIPTION" />
                  <title>Operation Description</title>
                  <text>
                    <list>
                      <item>Lieberman lid speculum placed between eyelids</item>
                      <item>Superior temporal incision performed in beveled three-stage maneuver</item>
                      ...
##Validation

There are two known validation errors found using the [Lantana CCDA validator](https://www.lantanagroup.com/validator/connectathon.jsp).
This tool was helpful for finding cda schema and ccda schematron violations with a single tool.

1. An unknown address is flagged as an error.  When using <addr nullFlavor="NI" />, the validator still complains 
about <addr> missing child elements.  [Others](https://www.projects.openhealthtools.org/sf/go/artf3450?returnUrlKey=1353511042478) 
seem to have experienced the same issue.
>If country is US, this addr SHALL contain exactly one [1..1] state, which SHALL be selected from ValueSet 2.16.840.1.113883.3.88.12.80.1 StateValueSet DYNAMIC (CONF:5402).
>Location:     /ClinicalDocument[1]  
>Test: 	(cda:recordTarget/cda:patientRole/cda:addr[cda:country='US' or cda:country='USA']) and cda:recordTarget/cda:patientRole/cda:addr/cda:state

2. As noted above, multiple procedure descriptions are used and scoped to the appropriate serviceEvent using the
Procedure Context template.  The validator complains because there are multiple procedure description elements where
only 1 is allowed. The schema will hopefully be relaxed in a future version to allow 1 or more procedure descriptions.
>SHALL contain exactly one [1..1] Procedure Description Section (templateId:2.16.840.1.113883.10.20.22.2.27) (CONF:9896).
>Location:     /ClinicalDocument[1]  
>Test: 	count(//cda:section[cda:templateId/@root='2.16.840.1.113883.10.20.22.2.27'])=1

The [SMART C-CDA Scorecard](http://ccda-scorecard.smartplatforms.org/) ranks each document at 72%.  100% of applicable best practices are met.  The documents are 
scored lower for missing structured vitals, which we suspect is a mistake because our documents do not include vitals. 
This tool was particularly helpful in finding coding inconsitencies, such as mismatched LOINC display names.



