GoogleCloudPrint
================

Google Cloud Print (GCP) For Java<br/>

now, it's not complete.<br/>
have problem about register printer (printer capabilities)<br/><br/>

document : <a href="https://developers.google.com/cloud-print/">https://developers.google.com/cloud-print/</a><br/><br/>

<h3>Example</h3>
<b>Connect to google cloud print</b>
```java
String email = "YOUR_GOOGLE_EMAIL";
String password = "YOUR_GOOGLE_PASSWORD";

GoogleCloudPrint cloudPrint = new GoogleCloudPrint();

//connect to google cloud print
cloudPrint.connect(email, password, "geniustree-cloudprint-1.0");
```
<b>Disconnect from google cloud print</b>
```java
cloudPrint.disconnect();
```
<b>Subscribe job</b> (listener job from google talk notification)
```java
cloudPrint.subScribeJob(new JobListener() {
    //
    @Override
    public void jobArrive(Job job, boolean success, String message) {
    
        //do something ...
    
    }
});    
```
```java
cloudPrint.subScribeJob(new JobListener() {
    //
    @Override
    public void jobArrive(Job job, boolean success, String message) {
        if (success) {
            try {
                LOG.debug("job arrive => {}", job);
                File directory = new File("C:\\temp\\cloudPrint");
                if (!directory.exists()) {
                  directory.mkdirs();
                }

                //download print file from google cloud print     
                String fileName = FilenameUtils.removeExtension(job.getTitle()) + ".pdf";
                File outputFile = new File(directory, fileName);
                 
                cloudPrint.downloadFile(job.getFileUrl(), outputFile);
        
 
 
                //do something...
                
                
                //get job information or printing option such as
                // - document collate 
                // - page output color 
                // - page orientation 
                // - page media size 
                // - duplex page 
                // - copies document
                // ...
                // response is XML format
                String ticketResponse = cloudPrint.getJobTicket(job.getTicketUrl());
                LOG.debug("ticketResponse => {}", ticketResponse);



                //update job
                ControlJobResponse response = cloudPrint.controlJob(job.getId(), JobStatus.DONE, 200, "success.");
                LOG.debug("control job response=> {}", response.isSuccess() + ", " + response.getMessage());
            } catch (CloudPrintException ex) {
                LOG.warn("Exception", ex);
            }
        } else {
            LOG.info("job arrive error message => {}", message);
        }
    }
});
```
<b>Submit job</b> (send job to google cloud print)
```java
//send job to target printer
//printer id = "810c1d39-981f-cd36-5fdc-951ea5e62613"

InputStream jsonInputStream = null;
InputStream imageInputStream = null;
try {
    jsonInputStream = Example.class.getResourceAsStream("/json/submitJobTicketRequest.json");
    imageInputStream = Example.class.getResourceAsStream("/testImage.png");
    byte[] content = IOUtils.toByteArray(imageInputStream);

    String jsonTicket = IOUtils.toString(jsonInputStream);
    Ticket ticket = gson.fromJson(jsonTicket, Ticket.class);

    String json = gson.toJson(ticket);
    LOG.debug("json => {}", json);
    
    //crate job
    SubmitJob submitJob = new SubmitJob();
    submitJob.setContent(content);
    submitJob.setContentType("image/png");
    submitJob.setPrinterId("810c1d39-981f-cd36-5fdc-951ea5e62613"); //*****
    submitJob.setTag(Arrays.asList("koalar", "hippo", "cloud"));
    submitJob.setTicket(ticket);
    submitJob.setTitle("testImage.png");

    //send job
    SubmitJobResponse response = cloudPrint.submitJob(submitJob);
    LOG.debug("submit job response => {}", response.isSuccess() + "," + response.getMessage());
    LOG.debug("submit job id => {}", response.getJob().getId());

    //control job
    //...
    //...
} catch (Exception ex) {
    LOG.warn("Exception", ex);
} finally {
    if (jsonInputStream != null) {
        try {
            jsonInputStream.close();
        } catch (IOException ex) {
            LOG.warn("Exception", ex);
        }
    }

    if (imageInputStream != null) {
        try {
            imageInputStream.close();
        } catch (IOException ex) {
            LOG.warn("Exception", ex);
        }
    }
}
```
<b>Search printer</b>
```java
//search all printers which name is "fax"
//PrinterStatus.ALL is all printer status 
// - ONLINE
// - UNKNOWN
// - OFFLINE
// - DORMANT

SearchPrinterResponse response = cloudPrint.searchPrinter("fax", PrinterStatus.ALL);
if (!response.isSuccess()) {
    return;
}

for (Printer printer : response.getPrinters()) {
    LOG.debug("printer => {}", printer);
}
```
<b>Get job</b>(print job)<br/>
get all jobs
```java
JobResponse response = cloudPrint.getJobs();
if (!response.isSuccess()) {
    return;
}

for (Job job : response.getJobs()) {
    LOG.debug("job response => {}", job);
}
```
get job of printer
```java
//printer id = "dc6929f5-8fdc-5228-1e73-c9dee3298445"

JobResponse response = cloudPrint.getJobOfPrinter("dc6929f5-8fdc-5228-1e73-c9dee3298445");
if (!response.isSuccess()) {
    return;
}

for (Job job : response.getJobs()) {
    LOG.debug("job response => {}", job);
}
```
<b>Delete printer</b>
```java
//printer id = "12280cbe-6486-2c98-c65a-22083bd18b5b"

DeletePrinterResponse response = cloudPrint.deletePrinter("12280cbe-6486-2c98-c65a-22083bd18b5b");
LOG.debug("delete printer response => {}", response.isSuccess() + ", " + response.getMessage());
```
<b>Share printer</b>
```java
//printer id = "dc6929f5-8fdc-5228-1e73-c9dee3298445"
//target email = "jittagorn@geniustree.co.th"

SharePrinterResponse response = cloudPrint.sharePrinter("dc6929f5-8fdc-5228-1e73-c9dee3298445", "jittagorn@geniustree.co.th");
LOG.debug("share printer message => {}", response.isSuccess() + ", " + response.getMessage());
```
develop by <b>jittagorn pitakmetagoon</b><br/>
java developer Thailand<br/>
work at Geniustree Co., Ltd<br/>
blog : <a href="http://na5cent.blogspot.com">na5cent.blogspot.com</a><br/>
email : jittagorn@geniustree.co.th
