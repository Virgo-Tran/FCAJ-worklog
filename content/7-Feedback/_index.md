---
title: "Sharing and Feedback"
date: 2024-01-01
weight: 7
chapter: false
pre: " <b> 7. </b> "
---

{{% notice note %}}
Below is my personal sharing and feedback regarding the entire internship experience during the First Cloud AI Journey program at AWS Vietnam.
{{% /notice %}}

Sharing my honest experience of the First Cloud AI Journey programme, so the team can build on what worked and change what did not.

### Overall Evaluation

**1. Working Environment**

The First Cloud AI Journey programme provides a professional, modern, and dynamic environment. Combining flexible online work with in-person sessions allowed me to cultivate self-discipline while staying closely connected with the cohort and team. The 13-week schedule was well-structured, with clear weekly milestones progressing from core account and infrastructure fundamentals (IAM, VPC, EC2, S3) to advanced topics (Containers, Serverless, IaC, and SageMaker). Having access to AWS sandbox accounts and official learning resources created ideal conditions for self-driven learning and steady progress.

**2. Support from Mentor / Team Admin**

Support from Mentors and Program Admins was one of the strongest highlights of the program. Despite busy schedules, Mentors were always prompt and dedicated in answering technical questions. Their detailed, constructive feedback on weekly reports and workshop architecture helped me spot knowledge gaps, fix security misconfigurations (such as IAM least privilege and VPC endpoints), and optimize system designs. The open communication style made me feel confident whenever seeking advice on complex issues.

**3. Relevance of Work to Academic Major**

My foundational knowledge from HUTECH (computer networks, relational databases, OS concepts, and web/API development) proved highly relevant and helped me absorb cloud concepts rapidly. Internship topics like AWS IAM, VPC, RDS, and DynamoDB directly built upon my university coursework. Furthermore, modern cloud-native practices such as Infrastructure as Code (AWS CDK/CloudFormation), Serverless Architecture, Container Orchestration (ECS/EKS), and AI services (Textract, SageMaker) provided valuable industry experience that complemented my academic degree.

**4. Learning & Skill Development Opportunities**

The 13-week curriculum offered a comprehensive learning path. Beyond theoretical concepts, I gained hands-on experience through numerous practical labs. The highlight of my internship was designing, documenting, and implementing the **Airport Information Management System** (AIMS) with an automated OCR document pipeline. This experience greatly sharpened my cloud architecture skills, bilingual technical documentation abilities, and end-to-end problem-solving mindset.

**5. Company Culture & Team Spirit**

The open, respectful, and supportive culture at AWS and within the AWS Study Group left a deep impression on me. Interns actively collaborated, shared study materials, and assisted each other during lab exercises. Participating in community events organized by the team expanded my professional network and provided valuable real-world insights from senior engineers and alumni.

**6. Internship Policies / Benefits**

The internship policies were very supportive. Flexible scheduling made it easy to balance internship tasks with university coursework at HUTECH. Generous access to AWS sandbox accounts, official learning resources, and sponsorship/support for AWS certification exams (such as AWS Certified Solutions Architect) were invaluable benefits for students. Program administration and progress tracking were consistently clear and professional.

---

### Additional Questions

**What did you find most satisfying during your internship?**

The most satisfying moment was seeing the automated document processing pipeline of the Airport Information Management System (AIMS) run successfully end-to-end. When a boarding pass image is uploaded to S3 via Presigned URL, the event triggers SQS, Textract extracts data, Lambda reconciles it against the PostgreSQL database on RDS, and SES sends an automated confirmation email — all backed by a Dead Letter Queue and CloudWatch alarms. Seeing an architecture I designed and implemented operate seamlessly was an incredibly rewarding milestone.

**What do you think should be improved for future interns?**

One minor improvement would be hosting additional hands-on workshops with Mentors on real-world CI/CD pipelines (CodePipeline, CodeBuild, CodeDeploy blue/green deployments) and live troubleshooting scenarios on real AWS infrastructure. This would give future cohorts deeper practical exposure to production deployment patterns and cost management.

**Would you recommend this internship to a friend? Why or why not?**

Yes, 100%. I would strongly recommend the First Cloud AI Journey to fellow students. It is an ideal program for motivated students passionate about Cloud and AI who want to experience world-class industry practices at AWS.

---

### Suggestions & Expectations

**Any suggestions to improve the internship experience?**

The program team could consider introducing short "Pair Programming" sessions or mini group challenges in the middle of the program to foster team bonding and peer learning among interns.

**Would you like to continue this program in the future?**

Yes, I would love to remain involved as an alumnus, Community Builder, or assistant mentor to support future cohorts of the AWS Study Group.

**Any other comments:**

I would like to express my sincere gratitude to the First Cloud AI Journey organization team, the Mentors, and AWS Vietnam for providing such an enriching internship experience that has significantly boosted both my technical capabilities and professional growth.
