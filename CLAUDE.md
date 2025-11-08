# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a **comprehensive Japanese tutorial repository** for JPy-DataReader, a Python library for fetching Japanese government statistics from e-Stat. The repository contains educational markdown files organized into 6 progressive chapters (beginner to advanced), totaling ~6,269 lines of documentation with 30 practice problems and solutions.

## Repository Structure

```
.
├── README.md                          # Tutorial homepage and overview
├── SUMMARY.md                         # Detailed table of contents
├── chapter01_basics.md               # Chapter 1: Beginner basics (893 lines)
├── chapter02_statslist.md            # Chapter 2: StatsListReader (1,108 lines)
├── chapter03_metainfo.md             # Chapter 3: MetaInfoReader (1,163 lines)
├── chapter04_statsdata.md            # Chapter 4: StatsDataReader (1,187 lines)
├── chapter05_advanced_analysis.md    # Chapter 5: Advanced analysis (815 lines)
└── chapter06_best_practices.md       # Chapter 6: Best practices (893 lines)
```

## Tutorial Architecture

### Chapter Progression
1. **Chapter 1** (Beginner): Installation, environment setup, e-Stat API key acquisition, basic 3 functions
2. **Chapters 2-4** (Intermediate): Deep dive into StatsListReader, MetaInfoReader, and StatsDataReader classes
3. **Chapters 5-6** (Advanced): Data analysis techniques, time series, regional comparison, error handling, performance optimization

### Consistent Chapter Format
Each chapter follows this structure:
- Detailed explanations of functionality
- Practical sample code with comprehensive Japanese comments
- 5 practice problems with hints (collapsible `<details>` sections)
- 5 complete model answers with detailed explanations
- Summary section and links to next chapter

## Content Guidelines

### When Editing Tutorial Content

**Language**: All tutorial content must be in Japanese. Code comments must also be in Japanese.

**Code Examples**: All Python code samples must include:
- Detailed Japanese comments explaining each section
- Proper error handling examples
- Real-world usage patterns
- References to JPy-DataReader's three main components:
  - `get_data_estat_statslist()` / `StatsListReader` - Search statistics tables
  - `get_data_estat_metainfo()` / `MetaInfoReader` - Fetch metadata/structure
  - `get_data_estat_statsdata()` / `StatsDataReader` - Retrieve actual data

**Practice Problems Format**:
```markdown
### 問題X: [Title]

**課題**: [Problem description]

<details>
<summary>ヒント</summary>

- [Hint 1]
- [Hint 2]

</details>
```

**Model Answers Format**:
```markdown
### 解答X: [Title]

\```python
"""
問題Xの模範解答: [Description]
"""
[Code with detailed Japanese comments]
\```

[Additional explanation if needed]
```

### Technical References

**JPy-DataReader Core Concepts**:
- **e-Stat API**: Requires API key (ESTAT_APP_ID) via environment variable
- **Three-layer access pattern**: List → MetaInfo → Data
- **Automatic pagination**: Handles datasets >100,000 records
- **Metadata structure**: Uses `@class`, `@code`, `@name`, `@level`, `@parentCode` for hierarchical classifications

**Sample Statistics Table IDs** used in examples:
- `"0003410379"` - Household survey (家計調査)
- `"00200524"` - National census code (国勢調査)

## Maintenance Tasks

### Adding New Content
- New chapters should follow the established 6-chapter progression model
- Maintain consistent problem count (5 per chapter)
- Ensure all code examples include comprehensive Japanese comments

### Updating Existing Chapters
- Preserve the existing problem/solution structure
- Update version history in README.md
- Keep cross-references between chapters accurate

### Version Updates
Update the version history section in README.md:
```markdown
### Version X.X.X (YYYY-MM-DD)
- ✅ [Change description]
```

## Related Resources

- **JPy-DataReader Repository**: https://github.com/kkawailab/jpy-datareader
- **e-Stat Official Site**: https://www.e-stat.go.jp/
- **e-Stat API Specification**: https://www.e-stat.go.jp/api/api-info/api-spec

## Git Workflow

This repository uses standard Git workflow with Japanese commit messages. Commits should be descriptive and use the format established in the initial commit.
