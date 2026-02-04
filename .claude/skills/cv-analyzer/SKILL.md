---
name: cv-analyzer
description: Analyze a CV/resume and identify strengths and weaknesses for a target job position. Use when user asks to analyze a CV, evaluate a candidate, or review a resume.
argument-hint: [path-to-cv]
---

# CV Analyzer

Analyze a CV against a target job position and produce a structured report of strengths and areas for improvement.

**Language adaptation:** Respond in the same language as the user's request.

## Process

1. **Collect information**
   - If CV path not provided as argument, ask for the file path
   - Ask for the target job position (title + context if available)

2. **Read and analyze the CV**
   - Read the CV file (PDF, DOCX, or text)
   - Extract key information: experience, skills, education, languages, certifications

3. **Evaluate against the position**
   - Compare candidate skills with typical job requirements
   - Assess experience level alignment

4. **Produce the report**

## Report Format

```markdown
## CV Analysis for [TARGET POSITION]

### Strengths
- [Strength 1 with justification]
- [Strength 2 with justification]
- ...

### Areas for Improvement
- [Weakness 1 with improvement suggestion]
- [Weakness 2 with improvement suggestion]
- ...

### Overall Fit
[Summary assessment of candidate-position alignment]

### Recommendations
- [Concrete actions to improve the CV]
```

## Evaluation Criteria

### Potential Strengths
- Directly relevant experience for the position
- In-demand technical skills
- Consistent career progression
- Valuable education and certifications
- Quantified and measurable achievements
- Industry-aligned keywords
- Clear and readable structure
- Relevant language proficiency

### Potential Weaknesses
- Missing key skills for the position
- Unexplained career gaps
- Insufficient or irrelevant experience
- Lack of concrete achievements
- Confusing or cluttered presentation
- Spelling errors or inconsistencies
- Outdated or irrelevant information
- No customization for the target position
