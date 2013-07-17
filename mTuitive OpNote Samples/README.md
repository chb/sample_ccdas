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
              <!-- We want to scope this section to the procedure that its subsections are documenting. -->
              <act moodCode="EVN" classCode="ACT">
                <templateId root="2.16.840.1.113883.10.20.6.2.5" />
                <!-- Procedure Context template to let us know exactly which procedure the sections in this scope document.-->
                <code code="29888" displayName="Arthroscopically aided anterior cruciate ligament repair/augmentation or reconstruction" codeSystem="2.16.840.1.113883.6.12" codeSystemName="CPT4" />
                <effectiveTime value="20060823222400" />
              </act>
              <title>Arthroscopically aided anterior cruciate ligament repair/augmentation or reconstruction (29888)</title>
              <component>
                <section>
                  <title>Laterality</title>
                  <text>
                    <list>
                      <item>Right</item>
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
                      <item>Tourniquet Used</item>
                      <item>Tourniquet inflated to 250mm Hg</iitem>
                      ...

