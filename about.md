---
layout: page
title: Syllabus
description: >-
    Course policies and information.
---

# Syllabus
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Course Description
Studies the structure, components, design, implementation, and internal operation of computer systems, focusing mainly on the operating system level. Reviews computer hardware and architecture including the arithmetic and logic unit, and the control unit. Covers current operating system components and construction techniques including the memory and memory controller, I/O device management, device drivers, memory management, file system structures, and the user interface. Introduces distributed operating systems. Discusses issues arising from concurrency and distribution, such as scheduling of concurrent processes, interprocess communication and synchronization, resource sharing and allocation, and deadlock management and resolution. Includes examples from real operating systems. Exposes students to the system concepts through programming exercises.

## Prerequisites
While the course does not have specific course prerequisites, students are expected to have reasonably strong experience in programming and a working knowledge of C.

Some understanding of key concepts in computer science at the undergraduate level, including computer architecture, operating systems, multi-threaded programming, networking, discrete mathematics, data structures, and algorithms, is presumed.

## Learning Outcomes
Upon completion of this course, the learner will be able to:

* trace the operation of context switching and program loading
* read part of an OS kernel and demonstrate an applied understanding of how abstract concepts apply to implementations and user programs
* use synchronization primitives (mutexes, semaphores, condition variables) to synchronize threads as demonstrated in a programming assignment
* describe operation of page tables and OS page faulting mechanisms for implementing demand allocation, demand loading, and copy-on-write 
* understand performance characteristics of hard drives; optionally demonstrate knowledge of RAID configurations via a programming assignment
* understand permissions and access control lists as implemented in Linux and Windows - optionally describe common software exploits.


# Course Materials

## Course Content

* Canvas will be used for assignment submission, grades, access to Zoom and class recordings, and class announcements. 
* The [course website](https://ahope-classes.github.io/neu-sea-spr26-cs5600/) will be used for all other materials
* The [course Github](https://github.com/ahope-classes/neu-sea-spr26-cs5600-labs) will be used for lab and in-class code. 

## Required Texts

This course will primarily utilize [Operating Systems: Three Easy Pieces](https://pages.cs.wisc.edu/~remzi/OSTEP/) by Remzi Arpaci-Dusseau and Andrea Arpaci-Dusseau. It is freely available online, and hard-copies are available for purchase. 

Other readings will be occasionally required and provided.  

## Communication

Communication between instructor, teaching assistants, and students is in English and is through

* E-mail via the Canvas distribution list
* Announcements posted on Canvas
* Notes and clarifications posted on Teams
* Private chats on Teams

Students are responsible for joining the class group on Teams and for ensuring that all email addresses are properly configured. Missing an email that is sent to Spam or not enrolling in Teams is not an excuse for any missed work or information.

You must check your Teams messages daily. We recommend that you install the Teams up on your mobile devices (phone, tablet) and turn on notifications so that you do not miss important messages or announcements.

To contact the instructor and the teaching assistant use Microsoft Teams as that ensures that communication is delivered and does not accidentally get filtered into “spam” or “junk” folders.

## Office Hours

* Office Hourse will be held via Zoom or Teams 
* Adrienne's OH will be Friday, 3-5pm
    * This time may change if it helps students better
    * OH may always be scheduled individually by contacting the instructor directly

## Schedule and Agenda


## Grading/Learning Assessment

* Homeworks (10): 40%
* Labs (5): 40%
* Lab presentations (1-3): 5%  (1-3 throughout the quarter, not for every lab)
* Final Exam: 15%

* The lowest 2 homework grades will be dropped. 

When completing assignments, you may use any material you wish from the web, slides, readings, lessons, etc., but you may not work with anyone or engage a third party (web site, tutoring service, someone else, etc.) unless explicitly stated in the assignment instructions. If you use any code or materials in an assignment, include a reference to the material.

## Work Submission

To receive full credit, all work for the course is expected to be completed by 11:59pm PT (Seattle Time) on the due date and must be submitted via Canvas. To foster fairness, all deadlines and due dates are to be strictly observed. Any graded work that is submitted after the due date is accepted with a 2.5% penalty for each day late until the solution is published or discussed in class, or until a date and time set by the instructor.

Mark your calendar with due dates as soon as a module is published and set reminders. Be particularly mindful of due times (AM versus PM) and time zones (all due times are PT/Seattle Time although your configuration of Canvas may be for a different time zone).

Submissions via email, as attachments to Canvas comments, Canvas Inbox messages, Teams, or through any manner other than via the Canvas assignment submission mechanism cannot be accepted under any circumstances. Any submissions received this way are not graded.

Check all submissions, including files, data sets, links, videos, and comments, prior and after submission to ensure that we have the correct work to grade. We grade what we receive. All submissions must be via Canvas and no submissions via email, Teams, direct messages, file attachments, or attachments to submission comments are accepted. Be sure to check that all links are publicly viewable when required.

When submitting, attach all required files or documents along with explanatory comments. Once an item has been graded, students will be able to view the grade and feedback via the grade book. Multiple submissions are accepted but only the last submission will be considered. After submission, check that your submission was successful.

All work is expected to be done in a professional and timely manner and must be written in English free of grammatical and spelling errors or a reduction in points will result. Using any third party material, ideas, or direct quotes must be properly cited. Anything directly copied must be placed in quotes and must be cited using either APA or MLA formats.

Be sure to frequently backup your work onto a cloud drive (OneDrive, Google Drive, Dropbox, S3, or similar; all students have a subscription for cloud storage on OneDrive and Google Drive through the University). **You may not backup to a public GitHub repository**. Alternatively, email your work and associated files to yourself frequently. Not meeting a deadline when work is lost because it was not backed up is not an acceptable excuse.

Be sure to have an alternative way to complete assignments in case of computer failure, e.g., borrow a computer, get a loaner from the University (Library or your Department), use a public computer on campus, use a secondary computer such as a ChromeBook, use posit.cloud, use repl.it, or a virtual machine on Azure, Google Cloud, or AWS, or similar.


## Grading Scale


| Score | Letter | GPA | Qualitative Meaning |
| ----- | ------ | --- |:------------------- |
|90% - 94.9% | A- |3.7| Excellent work with only a few but very minor flaws|
|87% - 89.9% |B+ |3.3 |Very good work with a several minor but immaterial flaws|
|84% - 86.9% |B |3.0 | Good and solid work having only immaterial flaws|
|80% - 83.9% |B-| 2.7 | Reasonably good work but significant flaws|
|77% - 79.9% |C+ | 2.3 | Adequate work; some significant flaws|
|73% - 76.9% | C | 2.0 | Adequate work; several significant flaws|
|70% - 72.9%| C- | 1.7 | Barely adequate work; several significant flaws|
|Less than 70% | F | 0.0 | Inadequate with significant flaws|


The course requires a 70% overall score to pass the course with two additional requirements to pass the course: (1) the average of all labs is at least 70% and (2) the final exam score is at least 70%. In other words, you cannot pass the course without getting a passing grade on the final exam and on the labs. So, if you get a 65% on the final and get a 100% on everything else you will not pass. 

If a student scores below 70% on the final exam, the student will be contacted by the instructor for an oral examination or additional written work, which, if of sufficient quality, will be used to mark the final exam as passed. For final grade calculation purposes, however, the originally achieved grade on the final exam will be applied.

## Use of AI Assistants and Tools

We understand that there are numerous resources available to assist you. In fact, we encourage that you use resources on the web and AI Assistants such as but not restricted to Copilot, Bard, Perplexity, Claude, or ChatGPT . While you may use AI assistants as needed we require that you let us know which ones you used and for what and that you add clear acknowledgments in your code, unless an assignment or work item explicitly prohibits the use of AI. Furthermore, you are responsible for knowing how your code works and you must be able to explain any code you copied or borrowed. So, while you may use assistants when permitted for inspiration, learning, and to get you started, you must learn from what you copied and you must understand what you copied.

## Collaboration

When completing graded work, you may use any material you wish from the course, web, slides, readings, lessons, etc., but you may not work with anyone or engage a third party (web site, tutoring service, someone else, AI Assistance, etc.) unless explicitly permitted in the instructions for a particular graded work item. If you use any code or materials in any graded work that you did not generate, include a reference to the material in your graded work. You are restricted to using coding mechanisms, techniques and methods that have been covered in the course up to the point when the graded work item is due. Any violation of these rules is serious and may result in a “F” for the course, a referral to OSCCR and the Khoury Academic Integrity Committee, and potential expulsion from the program and the University.

## Participation

Interaction occurs primarily through the University’s Microsoft Teams platform, although any graded peer group discussions are on Canvas. As part of each module, students are expected to:

* Complete all lessons and assigned work on time and within guidelines
* Be professional, courteous, and respectful in all communication
* Post their questions in the designated Teams channel
* Read all messages posted to Teams and Canvas daily
* Respond or comment on other students’ posts
* Join the live recitation sessions or watch the recording 

 ==Copying from the web or third parties without citation on any graded item is an academic integrity violations and results in a grade of 0 and a potential referral to OSCCR.==

Posts, questions, or responses on Teams are not graded and direct copying even without citation is allowed – but for those posts only.

## Code of Conduct
As a student in this course, students are expected conduct themselves at all times in a professional and respectful manner towards fellow students, instructors, teaching assistants, observers, graders, and anyone within or outside the course, College, University, and the community-at-large.

Specifically, students agree to:

* be speak in a respectful manner to instructors, teaching assistants, and graders
* resolve disputes over grades in a civil and constructive manner
* regard grades as a reflection of their mastery of the material, their commitment to timely submission or work, and the degree of mastery of the subject, and not as a token that is negotiated
* do all work on their own and to be students in the strictest sense of the word, i.e., one who studies and is an attentive and systematic observer
* not disparage others
* do all work in a timely fashion and when assigned
* to carefully read all assigned and provided material, study all lessons, and complete all graded and ungraded assignments, practices, laboratories, practicums, projects, reflections, quizzes, examinations, and any other exercises 
* use inclusive language and avoid actions that could be perceived as discriminatory or offensive
* embrace and respect the diversity of the class, which includes but is not limited to race, gender, age, sexual orientation, primary language, and cultural background
* attend all classes, recitations, except in cases of illness or emergencies, or to view recordings of the same as soon as being made available
* refrain from recording classes, recitations, chats, emails, or interactions, or sharing course materials without explicit permission by the instructor
* respond to surveys and polls, including any and all course evaluations

Egregious violations of the code of conduct may result in disciplinary actions, including reporting to University or College disciplinary committees and notation on the student’s academic record. Violations pertaining to learning may result in a reduction in scores or course grade.

## Language of Instruction
The language of instruction is (US) English. All materials are in English and all communication between and among students and instructors must be in English. All submissions and comments must be in English. All computer settings and messages must be in English.


## Academic Integrity Policy

The University views academic dishonesty as one of the most serious offenses that a student can commit and imposes appropriate punitive sanctions on violators. Here are some examples of academic dishonesty. While this is not an all-inclusive list, we hope this will help you to understand some of the things instructors look for.

While you may discuss your assignments with others in the current class, we require that all of your work submitted for grading be your own (unless specifically stated otherwise, e.g., group work) and not copied in whole or in part from anyone (including AI or AI Assistants). Students are expected to read and understand the Northeastern University Academic Honesty Policy (https://osccr.sites.northeastern.edu/academic-integrity-policy/). 

In general, unauthorized collaboration is any collaboration that has not been specifically authorized.

When there is concern whether a submission is based on unauthorized collaboration, use of AI, plagiarism, or use of third party material, the instructor may require the student to attend a meeting (in-person or virtual with camera turned on) and ask the student for an explanation or conduct an oral examination (viva voce). The purpose of the meeting or the viva voce does not have to be provided beforehand. When it becomes likely that a submission is not the student’s work, the instructor may replace the grade of the submission with a grade derived during the viva voce or a grade of 0 (or, if letter grades are used, an “F”), and, if egregious or on an exam, project, or practicum, reduce the student’s final grade or assign a failing grade for the course. 

The following is excerpted from the University’s policy on academic integrity; the complete policy is available in the Student Handbook.

* **Cheating** – intentionally using or attempting to use unauthorized materials, information or study aids in an academic exercise, or paying a third party, person, or firm to produce or edit work for the student
* **Fabrication** – intentional and unauthorized falsification, misrepresentation, or invention of any data, or citation in an academic exercise

* **Plagiarism** – intentionally representing the words, ideas, or data of another as one’s own in any academic exercise without quotation and without providing proper citation; that includes the use of online posts, blogs, and prior term’s submissions or solutions

* **Unauthorized collaboration** – instances when students submit individual academic works that are substantially similar to one another; while several students may have the same source material, the analysis, interpretation, and reporting of the data must be each individual’s independent work.

* **Participation in academically dishonest activities** – any action taken by a student with the intent of gaining an unfair advantage, including but not limited to copying work, submitting someone else’s work, falsifying work or files, hacking into systems, lying on a self-assessment, or submitting previously published solutions

* **Reusing Work** – submitting work submitted for credit or a grade in another course in the current or a prior term or at this or another institution

* **Facilitating academic dishonesty** – intentionally or knowingly helping or attempting to violate any provision of this policy; sending files to another student; allowing a student to use your own computer

* **Impersonation** – working on behalf of another students or allowing someone else to represent a student online, in discussion groups, in classes or sessions, or exams

* **Sharing** – sharing – whether for profit or not – materials, assignments, solutions, exams, exam question with another student in the class, outside the class, or in a future term, or on sharing sites, including but not limited to CourseHero, Quora, and Chegg

* **Sharing of URL** – sharing of classroom URLS (e.g., Zoom or Teams meetings) with anyone not currently enrolled in the course

* **Unannounced recording of conversations** – fully or partially recording audio, video, or chats in discussions, channels, or any online forum

* **Sharing of recordings or posts** – sharing portions or excerpts of any class recordings, posts, or other class communication with anyone not presently enrolled in the class

* **Sharing of login credentials** – sharing of username and/or password to allow anyone other than the students with those credentials to access any course resources

* **Misconduct** – any misconduct that is counter to the values of Northeastern University, e.g., social gatherings during a pandemic, not using masks in areas where required, use of inappropriate language or activities

* **AI Assistants** – any use of AI Assistants, where allowed and not specifically prohibited, must be acknowledged and students are responsible for ensuring the accuracy of any AI generated content. Directly copying answers generated by AI is never allowed.

Make certain that you are completely familiar with all policies, rules, and guidelines for this course. If in doubt, ask your instructor.
