# 🤖 AI Agent Instructions

> Role-based instructions for AI-assisted development across our organization. 

## 📚 Available Roles

| Role | Description | Link |
|------|-------------|------|
| **Core Instructions** | Shared foundation for all roles | [View](./CORE_INSTRUCTIONS.md) |
| **Frontend Developer** | React, Vue, Angular specialization | [View](./roles/frontend-developer.md) |
| **Backend Developer** | Node.js, Python, Go specialization | [View](./roles/backend-developer.md) |
| **Fullstack Developer** | End-to-end development | [View](./roles/fullstack-developer.md) |
| **Automation Test Engineer** | QA and test automation | [View](./roles/automation-test-engineer.md) |
| **DevOps Engineer** | Infrastructure and CI/CD | [View](./roles/devops-engineer.md) |
| **Mobile Developer** | iOS, Android, React Native, Flutter | [View](./roles/mobile-developer.md) |
| **Data Engineer** | Pipelines, ETL, data warehousing | [View](./roles/data-engineer.md) |
| **UI/UX Designer** | Design systems, user experience | [View](./roles/uiux-designer.md) |

## 🚀 How to Use

### With GitHub Copilot Chat

1. Copy the relevant role instructions
2. Start your prompt with the instructions
3. Or reference the file directly in Copilot Chat

```
@workspace Use the instructions from /docs/ai-instructions/frontend-developer.md
```

### With Custom GPTs or Claude Projects

1. Create a new custom GPT or Claude Project
2.  Paste the Core Instructions + Role-specific instructions into the system prompt
3. Use for all interactions related to that role

### In Your Repository

Add a `.github/copilot-instructions.md` file to your project repository:

```markdown
# Project-Specific AI Instructions

This project uses the Frontend Developer instructions from our central repository.

Additional project-specific rules:
- Use Tailwind CSS for styling
- Follow our component naming convention: PascalCase
- All API calls go through the `useApi` hook
```

## 🔄 Contributing

1. Fork this repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## 📋 Version History

| Version | Date | Changes |
|---------|------|---------|
| 3.1 | 2024-01 | Added UI/UX Designer, enhanced all roles |
| 3.0 | 2024-01 | Complete restructure with thinking modes |
| 2.0 | 2023-12 | Added reuse audit, verification plans |