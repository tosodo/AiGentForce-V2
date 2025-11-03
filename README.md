# AiGentForce-V2

AiGentForce-V2 is a scalable, role-based AI training platform for strategic consultants and transformation leads. It enables rapid deployment of Custom GPTs, workflow simulations, and branded onboarding assets—integrating with LangChain, n8n, and AIgentForce.io for seamless adoption.

## 🚀 Key Features

- **Role-Based AI Training**: Customizable training modules for different user roles and expertise levels
- **Custom GPT Deployment**: Rapid creation and deployment of branded AI assistants
- **Workflow Simulations**: Interactive scenario-based learning with real-time feedback
- **Branded Onboarding**: White-label onboarding experience with custom branding
- **LangChain Integration**: Advanced language model orchestration for complex workflows
- **n8n Workflow Automation**: No-code automation for multi-step processes
- **Document Analysis**: Advanced AI-powered document understanding and processing
- **Knowledge Base Management**: Centralized repository for training materials and documentation
- **Admin Dashboard**: Comprehensive administration and user management
- **Subscription Management**: Flexible pricing and subscription tier handling

## 📁 Project Structure

```
AiGentForce-V2/
├── app/                    # Main application logic and entry points
├── components/
│   └── ui/                # Reusable UI components
├── lib/                   # Utility functions and helpers
├── prisma/                # Database schema and migrations
├── types/                 # TypeScript type definitions
├── .github/               # GitHub configuration and workflows
│   ├── CONTRIBUTING.md    # Contribution guidelines
│   └── issue_template/    # Issue templates
├── docs/
│   ├── ARCHITECTURE.md    # System architecture documentation
│   ├── DEPLOYMENT.md      # Deployment guide
│   └── INTEGRATIONS.md    # Integration documentation
└── README.md              # This file
```

## 🛠️ Setup Instructions

### Prerequisites

- Node.js 16+ or Python 3.9+
- npm or yarn package manager
- PostgreSQL database
- API keys for integrated services

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/tosodo/AiGentForce-V2.git
   cd AiGentForce-V2
   ```

2. **Install dependencies**
   ```bash
   npm install
   # or
   yarn install
   ```

3. **Configure environment variables**
   ```bash
   cp .env.example .env.local
   # Edit .env.local with your configuration
   ```

4. **Setup database**
   ```bash
   npx prisma migrate dev
   ```

5. **Start development server**
   ```bash
   npm run dev
   ```

## 🔌 Integration Notes

### LangChain Integration
LangChain enables advanced language model orchestration for complex AI workflows. The integration allows for:
- Chain-of-thought prompting
- Multi-step reasoning
- Vector database integration
- Custom tool creation

See [INTEGRATIONS.md](docs/INTEGRATIONS.md#langchain) for detailed setup.

### n8n Workflow Automation
n8n provides no-code workflow automation capabilities:
- Trigger-based automation
- Multi-app integrations
- Conditional logic
- Error handling and retries

See [INTEGRATIONS.md](docs/INTEGRATIONS.md#n8n) for configuration details.

### Custom GPT Integration
Deploy branded ChatGPT assistants with custom knowledge bases:
- Fine-tuned model behavior
- Custom instructions and guidelines
- File upload capabilities
- Conversation memory

See [INTEGRATIONS.md](docs/INTEGRATIONS.md#custom-gpt) for implementation steps.

### AIgentForce.io Platform
Seamless integration with AIgentForce.io for:
- User authentication
- License management
- Analytics and reporting
- Multi-tenant support

See [INTEGRATIONS.md](docs/INTEGRATIONS.md#aigentforceio) for integration guide.

### Branded Onboarding
Customizable onboarding experience:
- White-label interface
- Custom colors and branding
- Role-based experience paths
- Progressive disclosure of features

See [INTEGRATIONS.md](docs/INTEGRATIONS.md#branded-onboarding) for customization options.

## 📚 Core Modules

### Admin Module
Comprehensive administration panel for managing:
- Users and roles
- Content and training materials
- System configuration
- Audit logs and compliance

### Subscription Management
Flexible subscription handling:
- Multiple pricing tiers
- Usage tracking
- Auto-renewal capabilities
- Invoice generation

### Knowledge Base
Centralized content repository:
- Document upload and management
- Full-text search
- Version control
- Access permissions

### Document Analysis
Advanced AI-powered analysis:
- Entity extraction
- Sentiment analysis
- Document classification
- Knowledge graph generation

## 📖 Documentation

- **[ARCHITECTURE.md](docs/ARCHITECTURE.md)** - Technical architecture and system design
- **[DEPLOYMENT.md](docs/DEPLOYMENT.md)** - Production deployment guide
- **[INTEGRATIONS.md](docs/INTEGRATIONS.md)** - Third-party service integrations
- **[CONTRIBUTING.md](.github/CONTRIBUTING.md)** - Contribution guidelines

## 🤝 Contributing

We welcome contributions! Please read our [CONTRIBUTING.md](.github/CONTRIBUTING.md) guide for details on our code of conduct and the process for submitting pull requests.

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🆘 Support

For support, please:
- Check the [Documentation](docs/)
- Review [GitHub Issues](https://github.com/tosodo/AiGentForce-V2/issues)
- Start a [Discussion](https://github.com/tosodo/AiGentForce-V2/discussions)

## 🚀 Quick Start

Visit [AIgentForce.io](https://aigentforce.io) to:
- Access pre-built templates
- Join the community
- Get support
- Explore integrations

---

**Version**: 2.0.0  
**Last Updated**: November 2024  
**Maintained by**: [tosodo](https://github.com/tosodo)
