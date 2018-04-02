# Introduction

Technology infrastructure is at a fascinating point in its history. Due to requirements for operating at tremendous scale, it has gone through a rapid disruptive change. The pace of innovation in infrastructure has been unrivaled except for the early days of computing and the internet. These innovations make infrastructure faster, more reliable, and more valuable.

The people and companies who have pushed the boundaries of infrastructure to its limits have found ways of automating and abstracting it to extract more business value. By offering a flexible, consumable resource, they have turned what was once an expensive cost center into a required business utility.

However, it is rare for utilities to provide financial value to the business, which means infrastructure is often ignored and seen as an unwanted cost. This leaves it with little time and money to invest in innovations or improvements.

How can such an essential and fascinating part of the business stack be so easily ignored? The business obviously pays attention when infrastructure breaks, so why is it so hard to improve?

Infrastructure has reached a maturity level that has made it boring to consumers.However, its potential and new challenges have ignited a passion in implementors and engineers.

Scaling infrastructure and enabling new ways of doing business have aligned engineers from all different industries to find solutions. The power of open source software (OSS) and communities are driven to help each other have caused an explosion of new concepts and innovations.

If managed correctly, challenges with infrastructure and applications today will not be the same tomorrow. This allows infrastructure builders and maintainers to make progress and take on new, meaningful work.

Some companies have surmounted challenges such as scalability, reliability, and flexibility. They have created projects that encapsulate patterns others can follow. The patterns are sometimes easily discovered by the implementor, but in other cases, they are less obvious.

In this book, we will share lessons from companies at the forefront of cloud native technologies to allow you to conquer the problem of reliably running scalable applications. Modern business moves very fast. The patterns in this book will enable your infrastructure to keep up with the speed and agility demands of your business. More importantly, we will empower you to make your own decisions about when you need to employ these patterns.

Many of these patterns have been exemplified in open source projects. Some of those projects are maintained by the Cloud Native Computing Foundation (CNCF). The projects and foundation are not the sole embodiment of the patterns, but it would be remiss of you to ignore them. Look to them as examples, but do your own due diligence to vet every solution you employ.

We will show you the benefits of cloud native infrastructure and the fundamental patterns that make scalable systems and applications. We’ll show you how to test your infrastructure and how to create a flexible infrastructure that can adapt to your needs.You’ll learn what is important and how to know what’s coming.

May this book inspire you to keep moving forward to more exciting opportunities, and to share freely what you have learned with your communities.

## Who Should Read This Book

If you’re an engineer developing infrastructure or infrastructure management tools, this book is for you. It will help you understand the patterns, processes, and practices to create infrastructure intended to be run in a cloud environment. By learning how things should be, you can better understand the application’s role and when you should build infrastructure or consume cloud services.

Application engineers can also discover which services should be a part of their applications and which should be provided from the infrastructure. Through this book, they will also discover the responsibilities they share with the engineers writing applications to manage the infrastructure.

Systems administrators who are looking to level up their skills and take a more prominent role in designing infrastructure and maintaining infrastructure in a cloud gateway can also learn from this book.

Do you run all of your infrastructure in a public cloud? This book will help you know when to consume cloud services and when to build your own abstractions or services.

Run a data center or on-premises cloud? We will outline what modern applications expect from infrastructure and will help you understand the necessary services to utilize your current investments.

This book is not a how-to and, outside of giving implementation examples, we’re not prescribing a specific product. It is probably too technical for managers, directors, and executives but could be helpful, depending on the involvement and technical expertise of the person in that role.

Most of all, please read this book if you want to learn how infrastructure impacts business, and how you can create infrastructure proven to work for businesses operating at a global internet scale. Even if you don’t have applications that require scaling to that size, you will still be better able to provide value if your infrastructure is built with the patterns described here, with flexibility and operability in mind.

## Why We Wrote This Book

We want to help you by focusing on patterns and practices rather than specific products and vendors. Too many solutions exist without an understanding of what problems they address.

We believe in the benefits of managing cloud native infrastructure via cloud native applications, and we want to prescribe the ideology to anyone getting started.

We want to give back to the community and drive the industry forward. The best way we’ve found to do that is to explain the relationship between business and infrastructure, shed light on the problems, and explain the solutions implemented by the engineers and organizations who discovered them.

Explaining patterns in a product-agnostic way is not always easy, but it’s important to understand why the products exist. We frequently use products as examples of patterns, but only when they will aid you in providing implementation examples of the solutions.

We would not be here without the countless hours' people have volunteered to write code, help others, and invest in communities. We love and are thankful for the people that have helped us in our journey to understand these patterns, and we hope to give back and help the next generation of engineers. This book is our way of saying thank you.

## Navigating This Book

This book is organized as follows:

- Chapter 1 explains what cloud native infrastructure is and how we got where we
  are.
- Chapter 2 can help you decide if and when you should adopt the patterns pre‐scribed in later chapters.
- Chapters 3 and 4 show how infrastructure should be deployed and how to write applications to manage it.
- Chapter 5 teaches you how to design reliable infrastructure from the start with testing.
- Chapters 6 and 7 show what managing infrastructure and applications look like.
- Chapter 8 wraps up and gives some insight into what’s ahead.
  If you’re like us, you don’t read books from front to back. Here are a few suggestions on broader book themes:
- If you are an engineer focused on creating and maintaining infrastructure, you should probably read Chapters 3 through 6 at a minimum.
- Application developers can focus on Chapters 4, 5, and 7, about developing infra‐structure tooling as cloud native applications.
- Anyone not building cloud native infrastructure will most benefit from Chapters1, 2, and 8.

## Online Resources

You should familiarize yourself with the Cloud Native Computing Foundation(CNCF) and projects it hosts by visiting the CNCF website. Many of those projects are used throughout the book as examples.

You can also get a good overview of where the projects fit into the bigger picture by looking at the CNCF landscape project (see Figure P-1).

Cloud native applications got their start with the definition of Heroku’s 12 factors. We explain how they are similar, but you should be familiar with what the 12 factors are(see http://12factor.net).

There are also many books, articles, and talks about DevOps. While we do not focus on DevOps practices in this book, it will be difficult to implement cloud native infra‐structure without already having the tools, practices, and culture DevOps prescribes.

![Cloud Native Landscape](https://raw.githubusercontent.com/cncf/landscape/master/landscape/CloudNativeLandscape_latest.png)

Figure P-1. CNCF landscape

## Acknowledgments

**Justin Garrison**

Thank you to Beth, Logan, my friends, family, and coworkers who supported us during this process. Thank you to the communities and community leaders who taught us so much and to the reviewers who gave valuable feedback. Thanks to Kris for making this book better in so many ways, and to you, the reader, for taking time to read books and improve your skills.

**Kris Nova**

Thanks to Allison, Bryan, Charlie, Justin, Kjersti, Meghann, and Patrick for putting up with my crap long enough for me to write this book. I love you and am forever grateful for all you do.