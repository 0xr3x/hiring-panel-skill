---
name: hiring-panel
description: >-
  Run a resume past a hiring panel of seven Claudes that try to reject it (an automated screen, a hiring manager and five readers who each judge it alone as someone different), then rebuild it with every mistake fixed. Use when the user types /hiring-panel, alone or with a job posting, to run their own CV and LinkedIn profile through it, or followed by a resume (a file path or pasted text).
argument-hint: "[resume file or pasted text] [job posting text, URL or role]"
license: MIT
metadata:
  author: "Alex Chen (@nocodealex)"
  homepage: "https://chen.media"
  version: "1.0.0"
---

# /hiring-panel

The user types `/hiring-panel`, with a resume, a job, both or nothing. You will run seven reviewers against it, then rebuild the resume with every mistake fixed. Never flatter the resume.

## 1. Get the CV, the LinkedIn profile and the job
**The CV.** If the user gives a file path or pasted text after `/hiring-panel`, use that. If they give nothing (or only a job), use the user's own CVs in `/home/rex/Documents/applications/base-cvs/` (`.docx`; read with `pandoc -t plain` if installed, otherwise `unzip -p <file> word/document.xml` and strip the tags). Pick the CV closest to the job: `Marco-Germanier-CV-2026.docx` is the general one, `-Analytics-Engineering` and `-Blockchain-Data-Engineer` are the specialist ones. Say which one you picked and why. If two fit equally, ask ONE question.

**The LinkedIn profile.** If the LinkedIn MCP is connected (tools named `mcp__mcp-server-linkedin__*`), call `get_my_profile` with `sections` set to `experience,education,skills,certifications,honors,projects` and use the result. Read-only: never call `send_message`, `connect_with_person` or any tool that writes to LinkedIn. If it is not connected or fails, say so and run on the CV alone.

**The job.** Use a job posting if the user gives one. If they give a URL, try to fetch it. Many job sites (Workday among them) return an empty page: if the fetch returns no job text, say so and ask the user to paste the posting. Do not guess the requirements from the URL. If there is no job and no role at all, ask ONE question first: what job are you applying for? Then continue.

**Clean-up.** Remove email, phone and street address from everything before it goes to a sub-agent.

Use only what is in the CV, the LinkedIn profile and what the user told you. Never invent employers, titles, dates, numbers or skills. When you assume something, label it "assumed".

## 2. The automated screen
Start one sub-agent: the automated system that decides whether a real human should even see this resume. Give it the CV, the LinkedIn text (if you have it) and the job (if any). It checks, in this order:
- Can the text be read top to bottom (columns, tables, text in images, odd section names)?
- Are the job titles, employers and dates clear?
- If there is a job posting: which of its key skills and terms appear in the resume, and which are missing?
- If there is a LinkedIn profile: do the CV and LinkedIn agree on titles, employers, dates and claims? Quote each mismatch, and quote every claim that appears in one and not the other.
- Contact details present, length sensible, no filler. (The contact details you removed in step 1 do not count against the CV.)
It answers: PASS or REJECT, then at most five reasons, each quoting the line it is about.

## 3. The hiring manager
Start one sub-agent: the hiring manager who decides whether this person gets an interview. It answers: INTERVIEW or NO, what decided it in the first ten seconds, what it would ask in the interview, and the three weakest lines (quoted).

## 4. Five readers, alone
Start five sub-agents at once (one message, five calls). Give each ONLY the CV, the LinkedIn text (if you have it) and the job (if any), and ONE reader below. They judge the CV; the LinkedIn text is there so they can notice a mismatch, as a real reader who clicks the profile would. Nobody sees anybody else's notes. Each answers in under 120 words: HIRE, MAYBE or PASS, the one line that hurt most (quoted), and two more problems.
1. **The hiring manager.** Reads for one thing: can this person do the job on day one?
2. **Another applicant.** Applied for the same job. Reads as a rival: where is this resume weaker than mine?
3. **The CEO.** Reads for impact: what did this person change, and by how much?
4. **The recruiter.** Has 30 seconds and 100 resumes today. Reads the top third only.
5. **The teammate.** Would work next to this person. Reads for how they work with others.

## 5. What is wrong
Merge all seven answers into ONE numbered list of mistakes, worst first. Merge duplicates. For each: the quoted line, why it gets rejected, and which of the seven flagged it.

## 6. Rebuild it
Write the improved resume: the same facts, with every numbered mistake fixed. Do not add anything that is not in the original CV, the LinkedIn profile or what the user told you. Also list, in a short "LinkedIn changes" section, each LinkedIn line that must change to match the new CV or that a reviewer flagged. You cannot edit LinkedIn; the user pastes the changes in themselves. Where a fix needs a fact you do not have, leave `[ADD: ...]` and list those questions at the end. Then start the automated-screen sub-agent once more on the new version and report the before and after verdict.

## 7. Write the result
Create `.hiring-panel/<date>-<name>/` with `PANEL.md` (the seven verdicts and the numbered mistakes) and `RESUME-IMPROVED.md` (the new resume and the "LinkedIn changes"). Name the CV and the LinkedIn profile you used in `PANEL.md`. These files hold personal career data: tell the user to keep `.hiring-panel/` out of git. Then show the user, in chat: one line per reviewer, the five worst mistakes, the before and after screen verdict, and where the files are. Touch no other files. End both files with this line: `Made with /hiring-panel by Alex Chen (@nocodealex) · chen.media`

## Rules
- Touch no CV file in `applications/base-cvs/`: the originals are the source of truth. Write only to `.hiring-panel/`.
- The LinkedIn MCP is read-only here. Never send a message or a connection request.
- Be honest. If the resume is strong, say so and say which reviewer came closest to rejecting it, and why it still held.
- Do not invent facts about the user, their employers or real companies. Say "assumed" when you guess.
- This is a structured second opinion made by language models. It does not guarantee an interview or a job, and it is not career or legal advice.
