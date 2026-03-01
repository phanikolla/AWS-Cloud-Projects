# Chapter 3 - Recipe Sharing Application - Complete Documentation Index

## 📊 Architecture Diagram
**File**: `chapter3_architecture.png`
- Professional PNG diagram
- Shows all AWS components
- Displays data flow and connections
- Generated via WSL + Graphviz

## 📖 Documentation Files

### 1. **README_ARCHITECTURE.md** ⭐ START HERE
Quick reference guide with:
- Overview of the application
- Key components summary
- API endpoints
- Deployment instructions
- Troubleshooting tips

### 2. **ARCHITECTURE.md**
Comprehensive documentation including:
- Detailed component descriptions
- Data flow diagrams (text-based)
- IAM & security configuration
- Deployment steps
- Technology stack
- Cost optimization tips

### 3. **ARCHITECTURE_DIAGRAM.txt**
ASCII art visualization showing:
- System architecture layout
- Component relationships
- Data flow paths
- Security layers
- Deployment flow

## 🔧 Tools & Scripts

### **generate_diagram_wsl.ps1**
PowerShell script for generating diagrams:
```powershell
.\generate_diagram_wsl.ps1 -OutputDir "." -DiagramName "my_diagram"
```
Requirements:
- WSL with Ubuntu installed
- PowerShell 5.0+

### **generate_diagram.py**
Python script for diagram generation:
```bash
python generate_diagram.py
```
Requirements:
- Python 3.13+
- diagrams package
- Working Graphviz installation

## 📋 Application Structure

```
chapter3/
├── code/
│   ├── backend/
│   │   ├── main.py              # FastAPI application
│   │   └── requirements.txt     # Python dependencies
│   ├── frontend/
│   │   ├── src/                 # React components
│   │   ├── package.json         # Node dependencies
│   │   └── vite.config.ts       # Build configuration
│   └── platform/
│       ├── ch3-http.yaml        # CloudFormation template
│       └── ch3-https-complete.yaml
├── INDEX.md                     # This file
├── README_ARCHITECTURE.md       # Quick reference
├── ARCHITECTURE.md              # Detailed docs
├── ARCHITECTURE_DIAGRAM.txt     # ASCII diagram
└── chapter3_architecture.png    # Visual diagram
```

## 🏗️ Architecture Overview

### Three-Tier Application
```
┌─────────────┐
│   Users     │
└──────┬──────┘
       │
   ┌───┴────────────────────┐
   │                        │
   ▼ HTTPS                  ▼ HTTP
┌──────────────┐      ┌──────────────┐
│ CloudFront   │      │ EC2 Instance │
│ + S3         │      │ (FastAPI)    │
└──────────────┘      └──────┬───────┘
                             │
                             ▼ Read/Write
                        ┌──────────────┐
                        │  DynamoDB    │
                        └──────────────┘
```

### Components
- **Frontend**: React + TypeScript on S3 + CloudFront
- **Backend**: FastAPI on EC2 with Nginx
- **Database**: DynamoDB (serverless)
- **Networking**: VPC with public subnet

## 🚀 Quick Start

### 1. View Architecture
```bash
# Open the diagram
open chapter3_architecture.png

# Or read the documentation
cat README_ARCHITECTURE.md
```

### 2. Deploy Application
```bash
aws cloudformation create-stack \
  --stack-name chapter3-recipe-app \
  --template-body file://code/platform/ch3-http.yaml
```

### 3. Access Application
- Frontend: `https://<cloudfront-domain>`
- Backend: `http://<ec2-public-dns>`

## 📚 API Reference

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/health` | Health check |
| GET | `/recipes` | List all recipes |
| POST | `/recipes` | Create recipe |
| DELETE | `/recipes/{id}` | Delete recipe |

## 🔐 Security Features

✅ IAM roles with least privilege
✅ Security groups restricting traffic
✅ S3 bucket with public access blocked
✅ CloudFront Origin Access Identity
✅ CORS configuration
✅ DynamoDB encryption

## 💰 Cost Estimation

| Service | Tier | Estimated Cost |
|---------|------|-----------------|
| EC2 | t3.micro | Free (first 12 months) |
| DynamoDB | On-demand | $0.25/million requests |
| S3 | Standard | $0.023/GB |
| CloudFront | Standard | $0.085/GB |

## 🐛 Troubleshooting

### Frontend not loading
→ Check `ARCHITECTURE.md` - Troubleshooting section

### API errors
→ Check EC2 instance status and IAM permissions

### DynamoDB access denied
→ Verify EC2 instance has correct IAM role

## 📞 Support Resources

- [FastAPI Docs](https://fastapi.tiangolo.com/)
- [React Docs](https://react.dev/)
- [AWS DynamoDB Guide](https://docs.aws.amazon.com/dynamodb/)
- [CloudFormation Reference](https://docs.aws.amazon.com/cloudformation/)

## 🔄 Diagram Generation

### Using WSL (Recommended)
```powershell
.\generate_diagram_wsl.ps1
```

### Using Docker
```bash
docker run -v $(pwd):/work python:3.13 bash -c "
  apt-get install graphviz
  pip install diagrams
  python /work/generate_diagram.py
"
```

### Using Online Tools
- [Draw.io](https://draw.io)
- [Lucidchart](https://www.lucidchart.com)
- [Miro](https://miro.com)

## 📝 File Descriptions

| File | Purpose | Size |
|------|---------|------|
| chapter3_architecture.png | Visual diagram | 82 KB |
| ARCHITECTURE.md | Detailed documentation | 4 KB |
| ARCHITECTURE_DIAGRAM.txt | ASCII visualization | 8 KB |
| README_ARCHITECTURE.md | Quick reference | 5 KB |
| INDEX.md | This file | 3 KB |

## ✅ Checklist

- [x] Architecture diagram generated
- [x] Documentation complete
- [x] API endpoints documented
- [x] Deployment guide provided
- [x] Security features listed
- [x] Troubleshooting guide included
- [x] Cost estimation provided
- [x] Tools and scripts created

## 🎯 Next Steps

1. **Review**: Read `README_ARCHITECTURE.md`
2. **Understand**: Study `ARCHITECTURE.md`
3. **Visualize**: View `chapter3_architecture.png`
4. **Deploy**: Use CloudFormation template
5. **Test**: Access the application
6. **Monitor**: Check AWS Console

---

**Last Updated**: January 31, 2026
**Status**: ✅ Complete
**Diagram Tool**: WSL + Graphviz
**Documentation**: Comprehensive
