# ═══════════════════════════════════════════════════════
# Fitly - Azure Deployment Guide
# Powered by NOVAPLM
# ═══════════════════════════════════════════════════════

## Architecture on Azure

```
┌─────────────────────────────────────────────────────────┐
│                    Azure Cloud                           │
│                                                          │
│  ┌──────────────┐     ┌──────────────────────────────┐  │
│  │  Azure CDN   │     │  Azure App Service (Backend) │  │
│  │  (Frontend)  │────▶│  Node.js + Express API       │  │
│  │  Static Web  │     │  Port 4000                   │  │
│  └──────────────┘     └──────────┬───────────────────┘  │
│                                  │                       │
│                       ┌──────────▼───────────────────┐  │
│                       │  Azure Database for           │  │
│                       │  PostgreSQL (Flexible Server) │  │
│                       └──────────────────────────────┘  │
│                                                          │
│  ┌──────────────┐     ┌──────────────────────────────┐  │
│  │ Azure Blob   │     │  Stripe Payment Gateway      │  │
│  │ Storage      │     │  (External)                   │  │
│  │ (Images)     │     │  NOVAPLM Platform Account     │  │
│  └──────────────┘     └──────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

## Step-by-Step Azure Deployment

### 1. Prerequisites
- Azure account with active subscription
- Azure CLI installed (`az login`)
- Node.js 18+ installed locally
- Stripe account for NOVAPLM

### 2. Create Azure Resources

```bash
# Set variables
RESOURCE_GROUP="fitly-rg"
LOCATION="eastus"
APP_NAME="fitly-api"
DB_SERVER="fitly-db"
DB_NAME="fitly"
DB_ADMIN="fitlyadmin"
DB_PASSWORD="YourSecurePassword123!"
STORAGE_ACCOUNT="fitlystorage"

# Create Resource Group
az group create --name $RESOURCE_GROUP --location $LOCATION

# Create PostgreSQL Flexible Server
az postgres flexible-server create \
  --resource-group $RESOURCE_GROUP \
  --name $DB_SERVER \
  --location $LOCATION \
  --admin-user $DB_ADMIN \
  --admin-password $DB_PASSWORD \
  --sku-name Standard_B1ms \
  --tier Burstable \
  --storage-size 32 \
  --version 15

# Create Database
az postgres flexible-server db create \
  --resource-group $RESOURCE_GROUP \
  --server-name $DB_SERVER \
  --database-name $DB_NAME

# Allow Azure services to connect
az postgres flexible-server firewall-rule create \
  --resource-group $RESOURCE_GROUP \
  --name $DB_SERVER \
  --rule-name AllowAzureServices \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 0.0.0.0

# Create Storage Account (for measurement images)
az storage account create \
  --resource-group $RESOURCE_GROUP \
  --name $STORAGE_ACCOUNT \
  --location $LOCATION \
  --sku Standard_LRS

# Create Blob Container
az storage container create \
  --account-name $STORAGE_ACCOUNT \
  --name fitly-uploads \
  --public-access blob

# Create App Service Plan
az appservice plan create \
  --resource-group $RESOURCE_GROUP \
  --name fitly-plan \
  --sku B1 \
  --is-linux

# Create Web App (Backend)
az webapp create \
  --resource-group $RESOURCE_GROUP \
  --plan fitly-plan \
  --name $APP_NAME \
  --runtime "NODE:18-lts"

# Create Static Web App (Frontend)
az staticwebapp create \
  --resource-group $RESOURCE_GROUP \
  --name fitly-frontend \
  --location $LOCATION
```

### 3. Configure Backend Environment Variables

```bash
DB_URL="postgresql://${DB_ADMIN}:${DB_PASSWORD}@${DB_SERVER}.postgres.database.azure.com:5432/${DB_NAME}?sslmode=require"

az webapp config appsettings set \
  --resource-group $RESOURCE_GROUP \
  --name $APP_NAME \
  --settings \
    DATABASE_URL="$DB_URL" \
    JWT_SECRET="$(openssl rand -base64 32)" \
    JWT_EXPIRES_IN="7d" \
    NODE_ENV="production" \
    PLATFORM_FEE_PERCENT="20" \
    STRIPE_SECRET_KEY="sk_live_your_stripe_key" \
    STRIPE_WEBHOOK_SECRET="whsec_your_webhook_secret" \
    FRONTEND_URL="https://fitly-frontend.azurestaticapps.net"
```

### 4. Deploy Backend

```bash
cd backend

# Install dependencies
npm install

# Generate Prisma client
npx prisma generate

# Run migrations
DATABASE_URL="$DB_URL" npx prisma migrate deploy

# Deploy to Azure
az webapp deployment source config-zip \
  --resource-group $RESOURCE_GROUP \
  --name $APP_NAME \
  --src ./deploy.zip

# Or use Git deployment:
az webapp deployment source config-local-git \
  --resource-group $RESOURCE_GROUP \
  --name $APP_NAME
```

### 5. Deploy Frontend

```bash
cd frontend

# For Static Web App, just deploy the index.html
# Option A: Azure Static Web Apps CLI
npm install -g @azure/static-web-apps-cli
swa deploy --app-location ./frontend --output-location .

# Option B: Via Azure Portal
# Upload index.html to Azure Static Web App
```

### 6. Configure Custom Domain (Optional)

```bash
# Add custom domain
az webapp config hostname add \
  --resource-group $RESOURCE_GROUP \
  --webapp-name $APP_NAME \
  --hostname api.fitly.com

az staticwebapp hostname set \
  --resource-group $RESOURCE_GROUP \
  --name fitly-frontend \
  --hostname www.fitly.com
```

### 7. Set up Stripe for NOVAPLM

1. Create Stripe account for NOVAPLM
2. Enable Stripe Connect (for paying tailors)
3. Set platform fee to 20%
4. Configure webhook endpoint: `https://fitly-api.azurewebsites.net/api/payments/webhook`

## Cost Estimate (Monthly)

| Service                          | SKU          | Est. Cost |
|----------------------------------|--------------|-----------|
| App Service (Backend)            | B1           | ~$13      |
| PostgreSQL Flexible Server       | B1ms         | ~$15      |
| Static Web App (Frontend)        | Free         | $0        |
| Blob Storage                     | Standard LRS | ~$1       |
| **Total**                        |              | **~$29/mo** |

Scale up as users grow. Move to Standard tier for production traffic.

## Monitoring

```bash
# Enable Application Insights
az monitor app-insights component create \
  --app fitly-insights \
  --location $LOCATION \
  --resource-group $RESOURCE_GROUP

# View logs
az webapp log tail --name $APP_NAME --resource-group $RESOURCE_GROUP
```
