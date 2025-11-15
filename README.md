Project Cerberus: K8s FinOps 자동화 프로젝트

"천방지축 날뛰는 애플리케이션에 자동화된 개목줄을 채우다"

1. 개요 (Overview)

본 프로젝트("Project Cerberus")는 쿠버네티스(k3s) 환경에서 발생하는 리소스 낭비와 정책 위반을 자동으로 감지하고, 방지하며, 교정하는 **'3중 제어 시스템'**을 구축하는 것을 목표로 합니다.

단순히 비용을 '모니터링'하는 것을 넘어, '천방지축 앱(Hellbeast)'이라는 고의적인 문제아 앱을 배포하여 다음과 같은 자동화 FinOps/SRE 파이프라인을 시연합니다.

방지 (Proactive Control): 잘못된 리소스 요청이나 보안 정책을 위반하는 앱의 배포를 사전에 차단합니다.

관찰 (Observation): 일단 배포된 앱이 얼마나 많은 리소스(CPU, Memory, Storage)를 '낭비'하고 있는지 정량화합니다.

교정 (Reactive Control): 감지된 낭비를 바탕으로, GitOps 워크플로우를 통해 자동으로 리소스를 최적화(교정)합니다.

2. 핵심 컨셉: 3-Headed Control System

이 프로젝트는 그리스 신화의 "케르베로스(Cerberus)"에서 이름을 따왔습니다. 클러스터의 비용과 안정성이라는 '지옥문'을 3개의 머리가 지키는 형상입니다.

[Head 1] The Observer (관찰) - Kubecost

클러스터에서 실행 중인 모든 워크로드의 리소스 사용량과 비용을 실시간으로 관찰하고 기록합니다.

requests와 실제 usage의 차이를 분석하여 '낭비'를 금전적 가치로 환산하고, 최적의 리소스 권장 사항을 API로 제공합니다.

[Head 2] The Gatekeeper (방지) - Kyverno

클러스터의 '문지기' 역할을 하는 Policy Engine입니다.

"메모리 1GiB 이상 요청 금지", "Root 권한 실행 금지", "특정 용량 이상의 PV 생성 금지"와 같은 '클러스터의 법'을 정의합니다.

이 정책을 위반하는 YAML이 제출되면, Admission Webhook 단계에서 배포 자체를 **거부(Block)**합니다. 이것이 첫 번째 '개목줄'입니다.

[Head 3] The Enforcer (교정) - ArgoCD + Automation Script

모든 배포는 Git 리포지토리를 통해서만 이루어집니다 (GitOps).

자동화 스크립트가 주기적으로 Kubecost(Head 1)의 API를 폴링하여 '최적화 권장 사항'을 가져옵니다.

스크립트가 Git 리포지토리의 배포 YAML을 자동으로 수정하고 커밋합니다.

ArgoCD가 이 변경 사항을 감지하고, '교정된' 버전의 앱을 클러스터에 자동으로 재배포합니다. 이것이 두 번째 '자동으로 당겨지는 목줄'입니다.

3. 등장인물 (The Cast)

The Hellbeast (천방지축 앱):

이 시연을 위해 특별히 제작된 Go 기반의 문제아 앱.

(정적 낭비): 실제 사용량(예: 10MiB)과 무관하게 과도한 리소스(예: 1GiB)를 requests합니다.

(스토리지 낭비): 불필요하게 큰 Persistent Volume(PV)을 요청합니다.

(정책 위반): Root 권한으로 실행되는 등 Kyverno의 정책에 위배되도록 설정됩니다.

(동적 낭비): 특정 API 호출 시 HPA를 발동시켜 파드를 늘렸다가, 트래픽이 끊겨도 불필요한 파드를 유지하여 비용을 발생시킵니다.

The Cluster (환경):

DigitalOcean VPS

K3s (경량 쿠버네티스)

4. 프로젝트 목표 및 시연 시나리오

목표

K8s 정책 엔진(Kyverno)을 통해 잘못된 배포를 사전에 차단하는 **'Proactive Control'**을 구현합니다.

모니터링(Kubecost)과 GitOps(ArgoCD)를 연동하여, 배포된 앱의 낭비를 자동으로 수정하는 'Reactive Control' 루프를 구축합니다.

FinOps가 단순한 '비용 관찰'이 아닌, 엔지니어링 자동화를 통해 '비용 최적화'로 이어지는 과정을 시연합니다.

시나리오

[1단계: 방지] Kyverno의 정책(예: Max 1GiB RAM)을 '초과'하는 "천방지축 앱"을 Git에 푸시 -> ArgoCD가 배포를 시도하지만 Kyverno에 의해 차단되어 'Sync Failed' 발생.

[2단계: 관찰] Kyverno 정책을 '통과'하지만(예: 0.9GiB RAM) 여전히 '낭비'가 심한 앱을 배포 -> Kubecost 대시보드에서 해당 앱이 '비효율' 상태로 감지되고 비용 낭비가 집계됨.

[3단계: 교정] 자동화 스크립트가 Kubecost의 권장 사항(예: 100MiB RAM)을 Git 리포지토리 YAML에 자동 커밋 -> ArgoCD가 변경을 감지하고 앱을 자동 재배포 -> Kubecost 대시보드에서 '낭비' 항목이 사라짐.

5. 기술 스택 (Tech Stack)

IaaS: DigitalOcean

Kubernetes: k3s

Cost Monitoring (Head 1): Kubecost

Policy Engine (Head 2): Kyverno

GitOps (Head 3): ArgoCD

Automation (Head 3): Shell Script (또는 Go Program)

Application: Go (Wasteful-Go-App)

(이하 생략: 설치 및 재현 방법, 향후 과제 등)
