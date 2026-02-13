---
name: create-drawio
description: Create .drawio architecture diagrams with proper AWS icons, containers, and clean layouts. Use when the user asks to create infrastructure diagrams, architecture diagrams, flowcharts, or any visual diagram in .drawio format.
---

# Draw.io Diagram Generator

Generate valid `.drawio` XML files that open in draw.io (diagrams.net) or the VSCode Draw.io extension.

## Process

1. **Understand the request**: What should the diagram show? Ask clarifying questions if needed.
2. **Explore the codebase** (if diagramming existing infrastructure): Read Terraform files, K8s manifests, docker-compose, etc.
3. **Plan the layout**: Decide which resources to include. Think about what a technical interviewer or architect would want to see.
4. **Generate the XML**: Write a valid `.drawio` file with proper structure, icons, containers, edges, and layout.
5. **Explain**: Briefly describe what the diagram shows and how to open/edit it.

## File Structure

Every `.drawio` file follows this structure:

```xml
<mxfile host="app.diagrams.net" modified="2026-01-01T00:00:00.000Z" agent="5.0" version="24.0.0" type="device">
  <diagram name="Page-1" id="page1">
    <mxGraphModel dx="1422" dy="900" grid="1" gridSize="10" guides="1" tooltips="1" connect="1" arrows="1" fold="1" page="1" pageScale="1" pageWidth="1600" pageHeight="900" math="0" shadow="0">
      <root>
        <mxCell id="0"/>
        <mxCell id="1" parent="0"/>
        <!-- All shapes, containers, and edges go here -->
      </root>
    </mxGraphModel>
  </diagram>
</mxfile>
```

**Required base cells**: `id="0"` (model root) and `id="1" parent="0"` (default layer) must always be present.

## Shapes (Vertices)

```xml
<mxCell id="myShape" value="Label Text"
        style="rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;fontSize=12;"
        vertex="1" parent="1">
  <mxGeometry x="200" y="150" width="120" height="60" as="geometry"/>
</mxCell>
```

- `id`: Unique identifier (use descriptive names like `eks`, `vpc`, `igw`)
- `value`: Display label (use `&#xa;` for line breaks)
- `vertex="1"`: Required for shapes
- `parent`: ID of container parent (`"1"` for top-level)
- Coordinates are **relative to parent** when nested in containers

## Edges (Connections)

```xml
<mxCell id="edge1" value="Label"
        style="edgeStyle=orthogonalEdgeStyle;rounded=1;orthogonalLoop=1;jettySize=auto;html=1;strokeColor=#666666;strokeWidth=2;endArrow=block;endFill=1;"
        edge="1" parent="1" source="shape1" target="shape2">
  <mxGeometry relative="1" as="geometry"/>
</mxCell>
```

- `edge="1"`: Required for edges
- `source`/`target`: IDs of connected shapes
- **Cross-container edges**: Set `parent="1"` — draw.io resolves absolute positions automatically
- Use `dashed=1` for management/config flows, solid for data/traffic flows

### Edge Styles

| Style | Use Case |
|-------|----------|
| `edgeStyle=orthogonalEdgeStyle;rounded=1;` | Clean right-angle routing (recommended) |
| `dashed=1;endArrow=open;endFill=0;` | Management / config connections |
| `strokeWidth=2;endArrow=block;endFill=1;` | Primary data flow |
| `curved=1;` | Long-distance connections |

## Containers (Grouping)

Containers use `container=1` in style. Children reference the container via `parent="containerId"`.

```xml
<mxCell id="myContainer" value="Container Label"
        style="...;container=1;collapsible=0;recursiveResize=0;pointerEvents=0;..."
        vertex="1" parent="1">
  <mxGeometry x="40" y="40" width="800" height="600" as="geometry"/>
</mxCell>

<!-- Child inside container -->
<mxCell id="child" value="Inside" style="..." vertex="1" parent="myContainer">
  <mxGeometry x="20" y="40" width="100" height="50" as="geometry"/>
</mxCell>
```

**Child coordinates are relative to parent's top-left corner.**

---

## AWS Icons Reference

### Resource Icon Template

Replace `FILL_COLOR` and `SERVICE_NAME`:

```
sketch=0;points=[[0,0,0],[0.25,0,0],[0.5,0,0],[0.75,0,0],[1,0,0],[0,1,0],[0.25,1,0],[0.5,1,0],[0.75,1,0],[1,1,0],[0,0.25,0],[0,0.5,0],[0,0.75,0],[1,0.25,0],[1,0.5,0],[1,0.75,0]];outlineConnect=0;fontColor=#232F3E;fillColor=FILL_COLOR;strokeColor=#ffffff;dashed=0;verticalLabelPosition=bottom;verticalAlign=top;align=center;html=1;fontSize=12;fontStyle=0;aspect=fixed;shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.SERVICE_NAME;
```

**Standard icon size**: 78x78 (or 68x68 for tighter layouts).

### Services by Category

**Compute (fillColor=#ED7100)**

| Service | resIcon | Terraform Resource |
|---------|---------|-------------------|
| EC2 | `ec2` | `aws_instance` |
| EC2 Instance | `instance3` | `aws_instance` |
| Lambda | `lambda` | `aws_lambda_function` |
| ECS | `ecs` | `aws_ecs_cluster`, `aws_ecs_service` |
| EKS | `eks` | `aws_eks_cluster` |
| Fargate | `fargate` | `aws_ecs_service` (FARGATE launch type) |
| ECR | `ecr` | `aws_ecr_repository` |
| Auto Scaling | `auto_scaling2` | `aws_autoscaling_group` |
| Elastic Beanstalk | `elastic_beanstalk` | `aws_elastic_beanstalk_application` |
| Batch | `batch` | `aws_batch_compute_environment` |
| Lightsail | `lightsail` | `aws_lightsail_instance` |
| Outposts | `outposts` | - |

**Networking (fillColor=#8C4FFF)**

| Service | resIcon | Terraform Resource |
|---------|---------|-------------------|
| VPC | `vpc` | `aws_vpc` |
| CloudFront | `cloudfront` | `aws_cloudfront_distribution` |
| Route 53 | `route_53` | `aws_route53_zone`, `aws_route53_record` |
| ELB | `elastic_load_balancing` | `aws_lb` |
| ALB | `application_load_balancer` | `aws_lb` (type=application) |
| NLB | `network_load_balancer` | `aws_lb` (type=network) |
| API Gateway | `api_gateway` | `aws_api_gateway_rest_api`, `aws_apigatewayv2_api` |
| NAT Gateway | `nat_gateway` | `aws_nat_gateway` |
| Internet Gateway | `internet_gateway` | `aws_internet_gateway` |
| Transit Gateway | `transit_gateway` | `aws_ec2_transit_gateway` |
| Direct Connect | `direct_connect` | `aws_dx_connection` |
| Global Accelerator | `global_accelerator` | `aws_globalaccelerator_accelerator` |
| VPC Peering | `vpc_peering` | `aws_vpc_peering_connection` |
| VPN Gateway | `vpn_gateway` | `aws_vpn_gateway` |
| Client VPN | `client_vpn` | `aws_ec2_client_vpn_endpoint` |
| App Mesh | `app_mesh` | `aws_appmesh_mesh` |
| Cloud Map | `cloud_map` | `aws_service_discovery_service` |
| PrivateLink/Endpoint | `endpoints` | `aws_vpc_endpoint` |
| Network ACL | `network_access_control_list` | `aws_network_acl` |

**Storage (fillColor=#7AA116)**

| Service | resIcon | Terraform Resource |
|---------|---------|-------------------|
| S3 | `s3` | `aws_s3_bucket` |
| EFS | `efs` | `aws_efs_file_system` |
| EBS | `elastic_block_store` | `aws_ebs_volume` |
| FSx | `fsx` | `aws_fsx_lustre_file_system`, `aws_fsx_windows_file_system` |
| Storage Gateway | `storage_gateway` | `aws_storagegateway_gateway` |
| Backup | `backup` | `aws_backup_plan` |
| S3 Glacier | `glacier` | `aws_glacier_vault` |
| Snowball | `snowball` | - |
| Snowmobile | `snowmobile` | - |

**Database (fillColor=#C925D1)**

| Service | resIcon | Terraform Resource |
|---------|---------|-------------------|
| RDS | `rds` | `aws_db_instance`, `aws_rds_cluster` |
| Aurora | `aurora` | `aws_rds_cluster` (engine=aurora) |
| DynamoDB | `dynamodb` | `aws_dynamodb_table` |
| ElastiCache | `elasticache` | `aws_elasticache_cluster` |
| DocumentDB | `documentdb_with_mongodb_compatibility` | `aws_docdb_cluster` |
| Neptune | `neptune` | `aws_neptune_cluster` |
| Redshift | `redshift` | `aws_redshift_cluster` |
| Timestream | `timestream` | - |
| QLDB | `qldb` | `aws_qldb_ledger` |
| DMS | `database_migration_service` | `aws_dms_replication_instance` |
| DAX | `dynamodb_dax` | `aws_dax_cluster` |

**Security (fillColor=#DD344C)**

| Service | resIcon | Terraform Resource |
|---------|---------|-------------------|
| IAM | `identity_and_access_management` | `aws_iam_role`, `aws_iam_user` |
| IAM Role | `role` | `aws_iam_role` |
| Secrets Manager | `secrets_manager` | `aws_secretsmanager_secret` |
| ACM | `certificate_manager` | `aws_acm_certificate` |
| WAF | `waf` | `aws_wafv2_web_acl`, `aws_waf_web_acl` |
| Cognito | `cognito` | `aws_cognito_user_pool` |
| KMS | `key_management_service` | `aws_kms_key` |
| Shield | `shield` | `aws_shield_protection` |
| GuardDuty | `guardduty` | `aws_guardduty_detector` |
| Inspector | `inspector` | `aws_inspector_assessment_target` |
| Macie | `macie` | `aws_macie2_account` |
| Security Hub | `security_hub` | `aws_securityhub_account` |
| Detective | `detective` | `aws_detective_graph` |
| CloudHSM | `cloudhsm` | `aws_cloudhsm_v2_cluster` |
| SSO | `single_sign_on` | `aws_ssoadmin_permission_set` |
| Directory Service | `directory_service` | `aws_directory_service_directory` |
| Firewall Manager | `firewall_manager` | `aws_fms_policy` |

**Management & Governance (fillColor=#E7157B)**

| Service | resIcon | Terraform Resource |
|---------|---------|-------------------|
| CloudWatch | `cloudwatch_2` | `aws_cloudwatch_log_group`, `aws_cloudwatch_metric_alarm` |
| CloudFormation | `cloudformation` | `aws_cloudformation_stack` |
| CloudTrail | `cloudtrail` | `aws_cloudtrail` |
| Config | `config` | `aws_config_config_rule` |
| Systems Manager | `systems_manager` | `aws_ssm_document` |
| Parameter Store | `systems_manager_parameter_store` | `aws_ssm_parameter` |
| Organizations | `organizations` | `aws_organizations_organization` |
| Control Tower | `control_tower` | `aws_controltower_control` |
| Service Catalog | `service_catalog` | `aws_servicecatalog_portfolio` |
| Trusted Advisor | `trusted_advisor` | - |
| License Manager | `license_manager` | `aws_licensemanager_license_configuration` |

**App Integration (fillColor=#E7157B)**

| Service | resIcon | Terraform Resource |
|---------|---------|-------------------|
| SQS | `sqs` | `aws_sqs_queue` |
| SNS | `sns` | `aws_sns_topic` |
| Step Functions | `step_functions` | `aws_sfn_state_machine` |
| EventBridge | `eventbridge` | `aws_cloudwatch_event_rule` |
| AppSync | `appsync` | `aws_appsync_graphql_api` |
| Amazon MQ | `mq` | `aws_mq_broker` |

**Analytics (fillColor=#8C4FFF)**

| Service | resIcon | Terraform Resource |
|---------|---------|-------------------|
| Athena | `athena` | `aws_athena_database` |
| Glue | `glue` | `aws_glue_job`, `aws_glue_crawler` |
| Kinesis | `kinesis` | `aws_kinesis_stream` |
| Kinesis Firehose | `kinesis_data_firehose` | `aws_kinesis_firehose_delivery_stream` |
| EMR | `emr` | `aws_emr_cluster` |
| MSK (Kafka) | `managed_streaming_for_kafka` | `aws_msk_cluster` |
| QuickSight | `quicksight` | `aws_quicksight_data_source` |
| Lake Formation | `lake_formation` | `aws_lakeformation_resource` |
| Elasticsearch/OpenSearch | `elasticsearch_service` | `aws_elasticsearch_domain` |
| Data Pipeline | `data_pipeline` | `aws_datapipeline_pipeline` |

**Machine Learning (fillColor=#01A88D)**

| Service | resIcon | Terraform Resource |
|---------|---------|-------------------|
| SageMaker | `sagemaker` | `aws_sagemaker_endpoint` |
| SageMaker Notebook | `sagemaker_notebook` | `aws_sagemaker_notebook_instance` |
| Comprehend | `comprehend` | `aws_comprehend_document_classifier` |
| Rekognition | `rekognition` | `aws_rekognition_collection` |
| Lex | `lex` | `aws_lex_bot` |
| Polly | `polly` | - |
| Transcribe | `transcribe` | `aws_transcribe_vocabulary` |
| Translate | `translate` | - |
| Textract | `textract` | - |
| Forecast | `forecast` | `aws_forecast_dataset` |
| Personalize | `personalize` | `aws_personalize_dataset` |

**Developer Tools (fillColor=#C925D1)**

| Service | resIcon | Terraform Resource |
|---------|---------|-------------------|
| CodePipeline | `codepipeline` | `aws_codepipeline` |
| CodeBuild | `codebuild` | `aws_codebuild_project` |
| CodeCommit | `codecommit` | `aws_codecommit_repository` |
| CodeDeploy | `codedeploy` | `aws_codedeploy_app` |
| Cloud9 | `cloud9` | `aws_cloud9_environment_ec2` |
| X-Ray | `x_ray` | `aws_xray_group` |
| CodeStar | `codestar` | `aws_codestarconnections_connection` |

**IoT (fillColor=#7AA116)**

| Service | resIcon | Terraform Resource |
|---------|---------|-------------------|
| IoT Core | `iot_core` | `aws_iot_thing` |
| IoT Greengrass | `iot_greengrass` | `aws_greengrass_group` |
| IoT Analytics | `iot_analytics` | `aws_iotanalytics_channel` |

---

## AWS Group/Container Styles

### Group Template

```
sketch=0;points=[[0,0],[0.25,0],[0.5,0],[0.75,0],[1,0],[1,0.25],[1,0.5],[1,0.75],[1,1],[0.75,1],[0.5,1],[0.25,1],[0,1],[0,0.75],[0,0.5],[0,0.25]];outlineConnect=0;gradientColor=none;html=1;whiteSpace=wrap;fontSize=12;fontStyle=1;container=1;pointerEvents=0;collapsible=0;recursiveResize=0;shape=mxgraph.aws4.group;grIcon=GROUP_ICON;strokeColor=STROKE;fillColor=FILL;verticalAlign=top;align=left;spacingLeft=30;fontColor=FONT;dashed=0;
```

### Group Types

| Group | grIcon | strokeColor | fillColor | fontColor |
|-------|--------|-------------|-----------|-----------|
| AWS Cloud | `mxgraph.aws4.group_aws_cloud_alt` | `#232F3E` | `none` | `#232F3E` |
| AWS Account | `mxgraph.aws4.group_account` | `#E7157B` | `none` | `#E7157B` |
| Region | `mxgraph.aws4.group_region` | `#00A4A6` | `none` | `#147EBA` |
| VPC | `mxgraph.aws4.group_vpc2` | `#8C4FFF` | `none` | `#AAB7B8` |
| Public Subnet | `mxgraph.aws4.group_security_group` + `grStroke=0` | `#7AA116` | `#F2F6E8` | `#248814` |
| Private Subnet | `mxgraph.aws4.group_security_group` + `grStroke=0` | `#147EBA` | `#E6F2F8` | `#147EBA` |
| Security Group | `mxgraph.aws4.group_security_group` | `#DD344C` | `none` | `#DD344C` |
| Auto Scaling Group | `mxgraph.aws4.group_auto_scaling_group` | `#ED7100` | `none` | `#ED7100` |

### Availability Zone (simple dashed rect)

```
fillColor=none;strokeColor=#147EBA;dashed=1;verticalAlign=top;fontStyle=1;fontColor=#147EBA;whiteSpace=wrap;html=1;container=1;pointerEvents=0;collapsible=0;recursiveResize=0;
```

### On-Premises / Corporate Datacenter

```
fillColor=none;strokeColor=#232F3E;whiteSpace=wrap;html=1;container=1;pointerEvents=0;collapsible=0;recursiveResize=0;verticalAlign=top;fontStyle=1;fontColor=#232F3E;
```

---

## Architecture Layout Patterns

### Diagram Hierarchy (outside-in)

1. **External actors** (Users, Internet, On-premises) — outside AWS Cloud
2. **AWS Cloud boundary** — outermost container
3. **Edge Services** (Route53, CloudFront, WAF, API Gateway, Cognito) — inside AWS Cloud, outside VPC
4. **VPC** — main network boundary
5. **Availability Zones** — optional, for multi-AZ visibility
6. **Subnets** (Public/Private) — inside VPC/AZ
7. **Security Groups** — optionally wrap resources inside subnets
8. **Resources** (EC2, EKS, RDS, etc.) — inside subnets
9. **Shared Services** (CloudWatch, ECR, KMS, S3) — inside AWS Cloud, outside VPC

### Edge Services (outside VPC, inside AWS Cloud)

These services sit at the "edge" of your architecture:
- Route 53 (DNS)
- CloudFront (CDN)
- WAF (Web Application Firewall)
- API Gateway
- Cognito (Authentication)
- AppSync (GraphQL)

### Shared Services Container

Group global services accessed by multiple resources:
- CloudWatch Logs
- ECR (Container Registry)
- KMS (Encryption)
- S3 (Storage)
- Secrets Manager
- SSM Parameter Store
- ACM (Certificates)

### Common Connection Patterns

| From | To | Arrow Style | Label |
|------|-----|-------------|-------|
| Users/Internet | Route53/CloudFront | Solid, reverse | HTTPS |
| Route53 | CloudFront/ALB | Solid | DNS |
| CloudFront | ALB/S3 | Solid | - |
| ALB/NLB | EC2/ECS/EKS | Solid | - |
| NAT Gateway | Internet Gateway | Solid | - |
| ECS/EKS | ECR | Dashed | Pull |
| Lambda | CloudWatch | Dashed | Logs |
| Any | S3/Secrets/KMS | Dashed | Access |
| Direct Connect | VPN Gateway/Transit Gateway | Solid | - |

---

## Layout Best Practices

### Spacing Guidelines

| Element | Value |
|---------|-------|
| AWS icon size | 68-78 x 68-78 px |
| Spacing between icons | 120-160 px horizontal, 100-140 px vertical |
| Container internal padding | 20-40 px |
| Container label space (top) | 40 px |
| Gap between containers | 20-30 px |

### Layout Rules

1. **Consistent IDs**: Use descriptive names (`vpc`, `pubSubnet`, `eks`, `e_users_igw`)
2. **Proper hierarchy**: AWS Cloud > Account/Region > VPC > AZ > Subnet > Security Group > Resources
3. **Align by tier**: Load balancers, app servers, databases in rows (left-to-right or top-to-bottom)
4. **Orthogonal edges**: Use `edgeStyle=orthogonalEdgeStyle;rounded=1;` for clean routing
5. **Fixed aspect**: Use `aspect=fixed` on AWS icons
6. **No HTML in labels**: draw.io renders HTML tags as literal text
7. **Grid alignment**: Snap to multiples of 10 when `gridSize=10`
8. **Multi-AZ**: Duplicate resources (NAT Gateway, EC2) in each AZ with suffix (~1, ~2)
9. **Security Groups**: Show as containers around related resources within subnets

### Interview-Ready Diagrams

When creating architecture diagrams for presentations or interviews:
- Show **main resources only** (VPC, subnets, compute, databases, key services)
- Skip granular details (route tables, individual policies, security group rules)
- Show **data flow** with solid arrows and **management/config** with dashed arrows
- Label connections with protocol or purpose (HTTPS, IRSA, Git Sync)
- Include external actors (Users, CI/CD, external APIs)
- Group Kubernetes workloads inside the cluster container
- Show multi-AZ deployment for production architectures

---

## Terraform Resource to Icon Mapping

When reading Terraform files, map resource types to icons:

```
aws_instance           → EC2 Instance
aws_lambda_function    → Lambda
aws_eks_cluster        → EKS
aws_ecs_service        → ECS (or Fargate if launch_type = "FARGATE")
aws_lb                 → ALB or NLB (based on load_balancer_type)
aws_db_instance        → RDS
aws_rds_cluster        → RDS/Aurora
aws_dynamodb_table     → DynamoDB
aws_s3_bucket          → S3
aws_sqs_queue          → SQS
aws_sns_topic          → SNS
aws_api_gateway_*      → API Gateway
aws_cloudfront_*       → CloudFront
aws_route53_*          → Route 53
aws_secretsmanager_*   → Secrets Manager
aws_kms_key            → KMS
aws_iam_role           → IAM Role
aws_cognito_*          → Cognito
aws_wafv2_*            → WAF
aws_vpc                → VPC container
aws_subnet             → Subnet container (check tags for public/private)
aws_security_group     → Security Group container
aws_nat_gateway        → NAT Gateway
aws_internet_gateway   → Internet Gateway
```

---

## Examples

Refer to the example files in `~/.claude/skills/create-drawio/examples/` for complete, working `.drawio` XML files:
- `aws-architecture.xml` — Full AWS architecture with VPC, subnets, EKS, and services
