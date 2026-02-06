# SplitDaBill

Bill splitting app for groups with shareable links.

## Features

- **Dual Mode System**
  - **Basic Mode**: Simple "I ate this, you paid that" with dropdown selections
  - **Advanced Mode**: Granular control with custom ratios and manual payments
- **Real-Time Collaboration**: Shareable session links with auto-save
- **Smart Split Calculation**: Automated settlement optimization with cover payment support
- **Color-Coded Participants**: 8 high-contrast colors with intelligent reuse
- **Live Validation**: Warns when total paid doesn't match bill total
- **Bank Details**: Add payment info for easy settlement

## Tech Stack

- **Backend**: FastAPI (Python)
- **Frontend**: HTML, CSS, JavaScript
- **Templates**: Jinja2
- **Database**: PostgreSQL (asyncpg), MongoDB

## Dependencies

```
fastapi
uvicorn
jinja2
pydantic
python-multipart
asyncpg
```

## Deployment

[Deployment Guide]()