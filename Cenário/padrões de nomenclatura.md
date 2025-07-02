## Recursos Principais

| Recurso                      | Prefixo | Formato                                    | Exemplo                       | Tamanho Máx. | Observações                        |
| ---------------------------- | ------- | ------------------------------------------ | ----------------------------- | ------------ | ---------------------------------- |
| **Resource Group**           | rg      | `rg-{workload}-{environment}-{region}`     | `rg-analytics-prod-eastus`    | 90           | Agrupa recursos relacionados       |
| **Virtual Network**          | vnet    | `vnet-{workload}-{environment}-{region}`   | `vnet-analytics-prod-eastus`  | 64           | Rede virtual principal             |
| **Subnet**                   | snet    | `snet-{workload}-{environment}-{az}`       | `snet-databricks-prod-01`     | 80           | Sub-rede dentro da VNet            |
| **Network Security Group**   | nsg     | `nsg-{workload}-{environment}-{region}`    | `nsg-databricks-prod-eastus`  | 80           | Regras de segurança de rede        |
| **Azure Databricks**         | dbw     | `dbw-{workload}-{environment}-{region}`    | `dbw-analytics-prod-eastus`   | 30           | Workspace do Databricks            |
| **Data Factory**             | adf     | `adf-{workload}-{environment}-{region}`    | `adf-etl-prod-eastus`         | 63           | Pipeline de dados                  |
| **SQL Server**               | sql     | `sql-{workload}-{environment}-{region}`    | `sql-dwh-prod-eastus`         | 63           | Servidor SQL                       |
| **SQL Database**             | sqldb   | `sqldb-{workload}-{environment}`           | `sqldb-warehouse-prod`        | 128          | Banco de dados SQL                 |
| **Azure Database for MySQL** | mysql   | `mysql-{workload}-{environment}-{region}`  | `mysql-app-prod-eastus`       | 63           | Servidor MySQL                     |
| **Container Registry**       | cr      | `cr{workload}{environment}{region}`        | `cranalyticsprodustus`        | 50           | Registro de containers (sem hífen) |
| **Container Instance**       | ci      | `ci-{workload}-{environment}-{region}`     | `ci-processor-prod-eastus`    | 63           | Instância de container             |
| **App Service**              | app     | `app-{workload}-{environment}-{region}`    | `app-portal-prod-eastus`      | 60           | Aplicação web                      |
| **Storage Account**          | st      | `st{workload}{environment}{region}`        | `stanalyticsprodustus`        | 24           | Conta de armazenamento (sem hífen) |
| **Event Grid Topic**         | egt     | `egt-{workload}-{environment}-{region}`    | `egt-events-prod-eastus`      | 50           | Tópico de eventos                  |
| **Event Hub Namespace**      | evhns   | `evhns-{workload}-{environment}-{region}`  | `evhns-streaming-prod-eastus` | 50           | Namespace do Event Hub             |
| **Key Vault**                | kv      | `kv-{workload}-{environment}-{region}`     | `kv-secrets-prod-eastus`      | 24           | Cofre de chaves                    |
| **Log Analytics Workspace**  | log     | `log-{workload}-{environment}-{region}`    | `log-monitoring-prod-eastus`  | 63           | Workspace de logs                  |
| **Application Insights**     | appi    | `appi-{workload}-{environment}-{region}`   | `appi-webapp-prod-eastus`     | 260          | Insights da aplicação              |
| **Azure Synapse**            | syn     | `syn-{workload}-{environment}-{region}`    | `syn-analytics-prod-eastus`   | 45           | Workspace Synapse                  |
| **Azure Function**           | func    | `func-{workload}-{environment}-{region}`   | `func-processor-prod-eastus`  | 60           | Azure Functions                    |
| **Cosmos DB**                | cosmos  | `cosmos-{workload}-{environment}-{region}` | `cosmos-catalog-prod-eastus`  | 44           | Banco NoSQL                        |
| **Service Bus Namespace**    | sb      | `sb-{workload}-{environment}-{region}`     | `sb-messaging-prod-eastus`    | 50           | Namespace Service Bus              |

## Componentes Internos

### Storage Account

| Componente     | Formato                     | Exemplo                         | Tamanho Máx. | Observações                  |
| -------------- | --------------------------- | ------------------------------- | ------------ | ---------------------------- |
| **Container**  | decidir em tempo de projeto | `raw-data-prod-v1`              | 63           | Apenas minúsculas e hífens   |
| **Blob Path**  | decidir em tempo de projeto | `sales/2024/06/27/data.parquet` | 1024         | Estrutura hierárquica        |
| **File Share** | `{workload}-{purpose        | `analytics-shared`              | 63           | Compartilhamento de arquivos |
| **Queue**      | `{purpose}`                 | `processing-queue`              | 63           | Fila de mensagens            |
| **Table**      | `{Entity}{Environment}`     | `CustomersProduction`           | 63           | Pascal case, sem hífens      |

### Databricks - Clusters e Componentes

| Componente             | Formato                        | Exemplo                | Observações             |
| ---------------------- | ------------------------------ | ---------------------- | ----------------------- |
| **Cluster Interativo** | `cl-interactive-{team}         | `cl-interactive-data   |                         |
| **Cluster Job**        | `cl-job-{workload}`            | `cl-job-etl-daily      | Para execução de jobs   |
| **Notebook**           | definir em tempo de projeto    |                        | Organização em pastas   |
| **Job**                | `pl-{frequency}-{workload}     | `pl-daily-etl-customer | Agendamento de execução |
| **Unity Catalog**      | `{organization}_{environment}` | `company_production`   | Catálogo de dados       |
| **Schema**             | `{domain}_{purpose}`           | `sales_analytics`      | Schema no Unity Catalog |

### Data Factory - Pipelines e Componentes

| Componente         | Formato                                   | Exemplo                 | Observações          |
| ------------------ | ----------------------------------------- | ----------------------- | -------------------- |
| **Pipeline**       | `pip_{workload}_{frequency}`              | `pl_customerETL_daily`  | Pipeline principal   |
| **Dataset**        | `ds_{source}_{entity}_{source ou target}` | `ds_SQL_customers`      | Fonte de dados       |
| **Linked Service** | `ls_{service}`                            | `ls_SQLcerver`          | Conexão com serviços |
| **Trigger**        | `trg_{frequency}_{workload}`              | `trg_daily_customerETL` | Gatilho de execução  |

### SQL Server - Bancos e Componentes

| Componente           | Formato                    | Exemplo               | Observações             |
| -------------------- | -------------------------- | --------------------- | ----------------------- |
| **Database**         | `{workload}_{environment}` | `DataWarehouse_PROD`  | Banco de dados          |
| **Schema**           | `{domain}`                 | `Sales`, `Marketing`  | Esquema lógico          |
| **Table**            | `{entity}`                 | `Customers`, `Orders` | Tabela de dados         |
| **View**             | `vw_{purpose}`             | `vw_CustomerSummary`  | Visão dos dados         |
| **Stored Procedure** | `sp_{action}_{entity}`     | `sp_Update_Customer`  | Procedimento armazenado |
| **Index**            | `IX_{table}_{columns}`     | `IX_Customers_Email`  | Índice de performance   |

### Event Grid - Tópicos e Componentes

| Componente             | Formato                   | Exemplo                  | Observações           |
| ---------------------- | ------------------------- | ------------------------ | --------------------- |
| **Custom Topic**       | `{domain}-{event-type}`   | `sales-order-created`    | Tópico customizado    |
| **System Topic**       | `{resource}-{event-type}` | `storage-blob-created`   | Tópico do sistema     |
| **Event Subscription** | `sub-{destination}`       | `sub-function-processor` | Subscrição de eventos |

### Key Vault - Segredos e Componentes

| Componente      | Formato                  | Exemplo                      | Observações           |
| --------------- | ------------------------ | ---------------------------- | --------------------- |
| **Secret**      | `{service}-{credential}` | `sql-connection-string-prod` | Segredo/credencial    |
| **Key**         | `{purpose}`              | `encryption-key-prod`        | Chave de criptografia |
| **Certificate** | `{service}`              | `webapp-ssl-prod`            | Certificado SSL       |
