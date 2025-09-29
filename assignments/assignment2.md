# Assignment 2
## Problem statement. 

### Problem domain

Job applications

The process is often repetitive since most applications ask the same thing but there are nicher (but still repetitive) things like cover letters and other application questions that are company-specific or are not included inside the resume which slow down the process. It's also (in this job market) not worth it usually to spend a lot of time on a job application, since auto-rejections are pretty common.


### Problem
Filling out a job application takes a long time but usually involves the same information, so users usually just end up filling the same things out over and over with minor adjustements. Cover letters and paragraph responses are also very time-costly, since they usually require company research and deeper thought.

### Stakeholders
1. Job applicants - directly affected because faster, higher-quality tools reduce stress and time spent on repetitive writing
2. Companies - more applicants may apply, and the applications they receive could be more polished
3. AI companies - the technology they produce as well as the quality will shape employer-applicant relationships



Evidence and comparables: a list of pieces of evidence that the problem is real, which will likely be citations of publicly available resources (but could also be observations that you have made about the extent or nature of the problem), and some comparables, being applications that address this or a related problem. Your list of evidence and comparables should have at least 5 entries, each with a title, an optional link, and a sentence or two explaining it.

1. [Simplify - really nice autofill thing, but can't autofill cover letters or more specific things](https://simplify.jobs) - It advertises that it is your "entire job search in one place", and is popularly used by a lot of my classmates. It addresses the problem of repetitive questions, and now automatically fills in job application slots that appear on your resume.

2. It takes four to eight months to find a job. [This is the survey.](https://www.thebalancemoney.com/how-long-does-it-take-to-find-a-job-2064245) This demonstrates that the job application process is very long, and applicants generally spend a lot of time sending out applications before they land a job.

3. [60% of job candidates stop applying midway through because it is taking too long](https://www.shrm.org/topics-tools/news/technology/study-job-seekers-abandon-online-job-applications#:~:text=According%20to%20CareerBuilder%2C%2060%20percent,of%20their%20length%20or%20complexity.). Not only do applicants waste time on applying partially, but companies also miss out on prospective candidates who might be more attracted to a different company's application process.

4. Each corporate job posting attracts ~250 resumes. Only 4-6 get called to [interview](https://blog.hiringthing.com/job-application-statistics#:~:text=Glassdoor%20found%20that%2C%20on%20average%2C%20each%20of%20their%20corporate%20job%20postings%20attracts%20approximately%20250%20resumes.%20Only%204%2D6%20will%20get%20called%20to%20interview%2C%20and%20one%20will%20get%20the%20job%20from%20those%20who%20apply.). This demonstrates how often, it is unlikely the application you are sending will pass the initial screen, and building on top of previous evidence that the job search usually lasts multiple months, it is unwise to spend too much time on each application since that's a lot of wasted time put towards applying instead of preparing for interviews.

5.  Grammerly cover letter generator (you have to put in your own prompt) [link](https://www.grammarly.com/ai/ai-writing-tools/cover-letter-generator). This demonstrates how there already exists tools that help with making the process faster--in particular cover letters.



## Application pitch. 

Name: Re.ply - the answer to your job search 

Motivation: Streamlines the job application process regarding paragraph prompts/cover letters to be faster and more streamlined

Key features: 

Note* the user is the applicant

1. Cover letter generator - Generate for prompts that require paragraph answers - like cover letters, company-specific questions. This helps mitigate the problem by researching the company for the user and generating a relevant draft, so the user can spend time editing instead of writing, researching, and editing. This impacts the user by cutting down time and increasing application experience. This impacts companies since they will likely receive more applications that have cover letters and polished responses. AI companies may receive higher traffic.

    Story: The user is submits information including a prompt, personal details to the generator, the generator responds with text that uses the personal details to match the prompt. 

2. Interactive Chat - User can have back and forth conversation with the LLM and give feedback, upload files, add additional requests, edit the output of the LLM with which the LLM provides a new edited version. This helps mitigate the problem of time spent on manually writing cover letters from scratch, and so the user can spend more time on editing as opposed as writing. AI Companies are impacted as this puts more weight on having a good AI model that is receptive to feedback and uploads. Companies are indirectly impacted by the output, since it will lead to higher quality cover letters.

    Story: After the user initially submits information and receives a generated text, the user submits feedback and additional information repetitively and also makes edits to the text as the LLM outputs.

3. Download as PDF - User can download the version of the cover letter that they're satisfied with to their device. Users are affected because they have a convenient way to just directly download the cover letter, as opposed to having to copy paste the output of an LLM to a document, save it as pdf, and then upload. Companies are indirectly affected since they may receive more applications with cover letters since the time needed for cover letters is lower. AI companies are not very affected by this feature, since downloading is not a concern of the generator.

    Story: After the user is satisfied with the output, the user can download as a pdf to their device and use it on a job application.


## Concept design.
Note: 3-5 concepts

1. 

concept Generator


purpose: an interactive answer that can be generated and modified by an LLM and modified, accepted, and copied by a user through feedback

principle: after the LLM generates answertype to the question, if there is user feedback, it will regenerate its output. Otherwise, the user can edit it or copy it for their use.

state 

a set of files FileStorage

a set of question String 
    a draft String
    a status String


    

actions

```
updateInput(file: FileStorage):
requires: file does not already exist in files
effect: adds file to set of files

generate(question: String, files: FileStorage): (draft: String)

requires there is no question with pending status
effects generate a draft to the question with status pending based off of current inputs

accept(question: String):

requires question to exist and draft status pending

effects set draft status to accepted

edit(d: String, content: String): (draft: String)

requires draft status is pending

effects return new draft with new content and same AnswerType 

feedback(d: String, feedback: string): (draft: String)

requires draft status to be proposed or edited

effects generate new text with updated content based off feedback and current inputs

```


2. concept PDFDownload

    purpose an interactive output of the generator that the user can modify or download as a PDF

    principle after a user prompts the generator and receives an output, the user can choose to edit it inline, download the text as a PDF

    state

    a set of filename String
        a content String

    actions
    ```
    setContent(content: String, filename: String): ()
    requires: filename does not already exist

    effect: maps filename to content 

    download(content: String, filename: String):()
    requires: filename exists and is mapped to content

    effect: downloads content with filename to local device

   

    download():

    ```

3. concept FileStorage

    purpose stores files that the user would like the generator to have as context while generating an answer to their question, such as writing style, skills, etc

    principle after a user uploads files, the output of the generator will utilize the information from the resume while answering the question/prompt, the user can also choose to remove files, and the generator will no longer use the information from that file

    state

    a set of Files
        a name String
        a content String

    actions
    ```
    upload(name: String, content: String): (File)
    requires: name does not already exist in Files
    effect: add new File to Files with name and content

    remove(name: String): (File)
    requires: name does exist in Files
    effect: remove file with name from Files

    files(): (Files)
    effect: return all Files 

    ```

synchronizations

```
sync generatorUsesFileStorage
    when FileStorage.upload(name, content): (file)
    then Generator.updateInput(file)

sync generatorRemovesFile
    when FileStorage.remove(name): (file)
    then Generator.removeInput(file)

sync generatorProducesPDF
    when Generator.generate(question, files): (draft)
    then PDFDownload.setContent(content: draft, filename: "draft.pdf")

sync generatorEditsPDF
    when Generator.edit(draft, content): (newDraft)
    then PDFDownload.setContent(content: newDraft, filename: "draft.pdf")

sync generatorFeedbackPDF
    when Generator.feedback(draft, feedback): (newDraft)
    then PDFDownload.setContent(content: newDraft, filename: "draft.pdf")

```

## UI Sketches.
![homepage](/assets/homepage.png)
The home page provides a simple interface for users to start interacting with the generator. At its core is a single text box where users can enter a prompt or question for the model. Alongside the text box is a button to upload files, allowing users to provide context, such as writing style, skills, or other reference material. Once uploaded, files can also be edited or removed, ensuring that the generator always uses the most relevant information when producing answers.

![chatinterface](/assets/chatinterface.png)
The chat interface functions similarly to a standard conversational interface like ChatGPT, allowing users to enter prompts and receive model-generated responses. Above the chat window, there is a files dropdown that displays all uploaded context files, which users can modify by adding or removing files. Each model-generated output is editable directly in the interface, giving the user control to refine content before accepting it. Beneath every response is a check mark that the user can click to mark the draft as “accepted,” updating the status of the question and finalizing the content.

![downloads](/assets/downloadscreen.png)
The download interface allows users to take any finalized text and save it locally as a PDF. Users can either edit the content inline before downloading or simply click a button to generate a PDF file with the text. The interface automatically handles mapping the text to a filename and triggers a local download to the user’s device, making it straightforward to save and share outputs from the generator.


## User journey. 

A user begins by identifying a problem: they need to generate a response to a question or prompt that reflects their personal or professional context, such as writing style, skills, or past experiences. They want the answer to be tailored, high-quality, and something they can save for later use as a PDF.

The user opens the application and is presented with the home page, which includes a simple textbox for entering their prompt and a button to upload files. They use the file upload feature to add relevant context documents, such as a resume or notes, and may edit or remove files to ensure the generator has the most accurate information.

Next, the user types their question or prompt into the textbox and submits it. The generator produces a draft response based on the uploaded files and displays it in the chat interface. In this interface, the user can review the output and make inline edits to refine the content. If needed, they can also provide feedback to the generator, prompting it to regenerate the draft. Once satisfied, the user clicks the check mark under the output to accept the draft, marking the question as complete.

Finally, the user navigates to the download interface. Here, they have the option to make any last edits before saving the content. They then click a button to download the text as a PDF, which is saved locally on their device. By the end of this journey, the user has a finalized, context-aware response that incorporates their uploaded files and is preserved as a PDF, ready for sharing or personal use.