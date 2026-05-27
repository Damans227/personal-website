---
title: "AWS Machine Learning Services: What Each One Does"
date: 2026-05-18T14:00:00Z
draft: false
tags:
  - AWS
  - Machine Learning
  - AI
---

AWS offers a family of ready-made machine learning services that handle a single, well-defined task each: recognize faces, transcribe speech, translate text, recommend products, and so on. You call an API and get a result — no model training, no GPUs, no data science required. SageMaker sits alongside them for the cases where you do want to build your own model.

This is a tour of what each service is, what it is for, and when to reach for it.

---

## The mental map

```
┌─────────────────────────────────────────────────────────┐
│  VISION (images, documents)                             │
│  • Rekognition  (objects, faces, scenes in images/video)│
│  • Textract     (text & data from scanned documents)    │
├─────────────────────────────────────────────────────────┤
│  SPEECH (audio in, audio out)                           │
│  • Transcribe   (speech → text)                         │
│  • Polly        (text → speech)                         │
├─────────────────────────────────────────────────────────┤
│  LANGUAGE (text understanding)                          │
│  • Translate    (translation across languages)          │
│  • Comprehend   (NLP — sentiment, entities, topics)     │
├─────────────────────────────────────────────────────────┤
│  CONVERSATIONAL                                         │
│  • Lex          (chatbots, voice bots)                  │
│  • Connect      (cloud contact center, pairs with Lex)  │
├─────────────────────────────────────────────────────────┤
│  SEARCH & RECOMMENDATIONS                               │
│  • Kendra       (ML-powered document search)            │
│  • Personalize  (real-time recommendations)             │
├─────────────────────────────────────────────────────────┤
│  BUILD YOUR OWN                                         │
│  • SageMaker    (full ML platform for data scientists)  │
└─────────────────────────────────────────────────────────┘
```

---

## Amazon Rekognition

Rekognition finds **objects, people, text, and scenes in images and videos** using machine learning. It also does facial analysis and facial search — useful for user verification and people counting — and can match against a database of "familiar faces" or against known celebrities.

Typical use cases:

- **Labeling** — auto-tag images by what is in them.
- **Content moderation** — flag unsafe or inappropriate content.
- **Text detection** — pull text out of an image.
- **Face detection and analysis** — gender, age range, emotions.
- **Face search and verification.**
- **Celebrity recognition.**
- **Pathing** — track how a person or object moves through a video, for example in sports analysis.

---

## Amazon Transcribe

Transcribe converts **speech to text**. Under the hood it uses a deep learning process called **automatic speech recognition (ASR)** to do it quickly and accurately.

A few features worth knowing:

- **Redaction** automatically removes Personally Identifiable Information (PII) from the transcript.
- **Automatic Language Identification** handles multi-lingual audio.

Common uses:

- Transcribing customer service calls.
- Generating closed captions and subtitles.
- Producing searchable metadata for a media archive.

---

## Amazon Polly

Polly is the mirror image of Transcribe: it turns **text into lifelike speech** using deep learning. It is what you reach for when you want an application that talks — voice assistants, IVR, accessibility features, narrated content.

---

## Amazon Translate

Translate provides **natural and accurate language translation**. It lets you localize content — websites, applications, documents — for international users, and translate large volumes of text without writing your own translation pipeline.

---

## Amazon Lex & Connect

These two often appear together because they build the same thing — a voice-driven application — at different layers.

**Amazon Lex** is the same technology that powers Alexa. It combines:

- **Automatic Speech Recognition (ASR)** to convert speech to text.
- **Natural Language Understanding** to recognize the *intent* behind that text.

It is what you use to build chatbots and call-center bots.

**Amazon Connect** is a **cloud-based virtual contact center**. It receives calls, lets you design contact flows, and integrates with CRMs and other AWS services. AWS markets it as roughly 80% cheaper than traditional contact-center solutions, with no upfront payments.

Together they look like this:

```
Phone Call                 call         stream        invoke       schedule
"Schedule an    ──►  Connect  ──►  Lex (intent  ──►  Lambda  ──►  CRM
 Appointment"                       recognized)
```

Connect receives the call, Lex figures out what the caller wants, Lambda executes the action, and the CRM is updated.

---

## Amazon Comprehend

Comprehend is the **natural language processing (NLP)** service — fully managed and serverless. It uses machine learning to find insights and relationships in text:

- The **language** of the text.
- **Key phrases, places, people, brands, and events.**
- **Sentiment** — how positive or negative the text is.
- **Tokenization and parts of speech.**
- **Topic modeling** — automatically organize a collection of text files by topic.

Sample uses:

- Analyzing customer emails to see what leads to positive or negative experiences.
- Grouping articles by topics that Comprehend discovers on its own.

---

## Amazon SageMaker

The previous services are all *pre-trained* — you call them and get an answer. SageMaker is the opposite: it is a **fully managed platform for developers and data scientists to build, train, and deploy their own ML models**.

Without SageMaker, doing the whole ML lifecycle in one place — provisioning servers, labeling data, training, tuning, deploying — is awkward. SageMaker rolls it into a single service. The simplified flow:

```
Historical data ──label──► labeled data ──build──► ML model ──train & tune
                                                       │
                                                       ▼
                                                  New data ──apply──► Prediction
```

If you have a custom problem and want to train a model on your own data, SageMaker is the place. If a problem fits Rekognition, Transcribe, Comprehend, or one of the other purpose-built services, use those instead — they are simpler and you don't need to train anything.

---

## Amazon Kendra

Kendra is a **fully managed document search service powered by machine learning**. It can extract answers from inside documents — text, PDFs, HTML, PowerPoint, Word, FAQs — and answer questions in natural language.

It connects to a wide range of data sources (S3, RDS, Google Drive, SharePoint, OneDrive, Salesforce, ServiceNow, and more), indexes them, and exposes a unified search interface.

Two features worth calling out:

- **Incremental learning** — it learns from user interactions and feedback to promote preferred results.
- **Manual tuning** — you can boost importance, freshness, or specific results yourself.

Picture an employee typing "Where is the IT support desk?" into a search bar and Kendra answering "1st floor" by pulling it from an internal wiki.

---

## Amazon Personalize

Personalize is a **managed service for real-time personalized recommendations** — the same technology that powers Amazon.com. You feed it user and item data; it returns recommendations through a customized API.

It plugs into existing websites, apps, SMS, and email systems. The selling point: you can ship recommendations in days rather than spending months building, training, and deploying your own model.

```
Amazon S3 ──read data──►
                         Amazon Personalize ──Customized API──► Websites, apps,
Personalize API ──stream─►                                       SMS, emails
```

Typical use cases are retail stores, media, and entertainment — anywhere "users who liked X also liked Y" creates value.

---

## Amazon Textract

Textract automatically **extracts text, handwriting, and data from scanned documents** using AI and ML. It handles PDFs and images, and it understands the structure of forms and tables — so it doesn't just give you a wall of text, it gives you fields and values.

A scanned driver's license, for example, comes back as JSON:

```json
{
  "Document ID": "123456789-005",
  "Name": "...",
  "SEX": "F",
  "DOB": "23.05.1997"
}
```

Common domains:

- **Financial services** — invoices, financial reports.
- **Healthcare** — medical records, insurance claims.
- **Public sector** — tax forms, ID documents, passports.

---

## Decision shortcuts

| Need | Service |
|------|---------|
| Detect objects, faces, or text in images/video | **Rekognition** |
| Convert speech to text | **Transcribe** |
| Convert text to speech | **Polly** |
| Translate text between languages | **Translate** |
| Build a chatbot or voice bot | **Lex** |
| Run a cloud-based contact center | **Connect** |
| NLP — sentiment, entities, topics | **Comprehend** |
| Build and train your own ML model | **SageMaker** |
| Search across internal documents | **Kendra** |
| Real-time product recommendations | **Personalize** |
| Pull data out of scanned forms or PDFs | **Textract** |

---

## Summary

- AWS's ML services split cleanly into two camps: **purpose-built APIs** that solve one well-defined task each, and **SageMaker** for building your own models.
- For vision, reach for **Rekognition** and **Textract**. For speech, **Transcribe** and **Polly**. For language, **Translate** and **Comprehend**. For conversation, **Lex** with **Connect** on top.
- **Kendra** is search, **Personalize** is recommendations, and **SageMaker** is the escape hatch when none of the pre-trained services fit.
- Pick by the task you want to solve — the service names are descriptive enough that the right one is usually obvious once you know they exist.

---
