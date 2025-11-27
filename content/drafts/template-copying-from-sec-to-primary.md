1. Upload template to CloudStack
   └── Template goes to secondary storage
   └── Stored at /export/secondary/template/...

2. First VM created using that template
   └── CloudStack checks: "Is template on primary?"
   └── No → SSVM copies template from secondary → primary
   └── Template now on /export/primary/template/...

3. VM disk created
   └── Clone/copy of template on primary
   └── VM boots

4. Second VM created using same template
   └── CloudStack checks: "Is template on primary?"
   └── Yes → skip copy
   └── VM disk created directly
