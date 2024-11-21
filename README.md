# S3와 CloudFront 성능 비교 분석

이 문서는 Amazon S3와 CloudFront CDN을 사용했을 때의 성능 차이를 비교 분석합니다.

## 다이어그램
<img src="https://github.com/user-attachments/assets/24522b4e-0ac0-4785-a698-0444bc3a48e1" width="800" height="400" alt="다이어그램">

## 배포 워크플로우

- 이 GitHub Actions 워크플로우는 코드 변경 사항이 `main` 브랜치에 대한 Pull Request가 닫힐 때 자동으로 실행되며, 수동으로도 트리거할 수 있습니다. 
- 이 워크플로우는 Node.js 애플리케이션을 빌드하고, AWS S3에 배포하며, CloudFront 캐시를 무효화하는 과정을 포함합니다.

### 주요 단계
- 이 배포 워크플로우는 `Pull Request가 병합`된 경우에만 실행됩니다.

1. **Checkout repository**:
   - `actions/checkout@v4`를 사용하여 현재 저장소의 코드를 체크아웃합니다. 
   - 이 단계는 이후 단계에서 코드에 접근할 수 있도록 합니다.

2. **Setup Node.js**:
   - `actions/setup-node@v4`를 사용하여 Node.js 환경을 설정합니다. 
   - Node.js 버전 20을 사용하며, npm 패키지 캐싱을 자동으로 처리합니다.
   - `cache: 'npm'` 옵션을 통해 패키지 설치 시간을 단축합니다.

3. **Install packages**:
   - `npm ci` 명령을 실행하여 필요한 Node.js 패키지를 설치합니다. 
   - 이 명령은 `package-lock.json`에 정의된 정확한 버전의 패키지를 설치합니다.

4. **Build**:
   - `npm run build`를 실행하여 애플리케이션을 빌드합니다. 
   - 이 단계에서 애플리케이션의 최종 결과물이 생성됩니다.

5. **Configure AWS credentials**:
   - `aws-actions/configure-aws-credentials@v4`를 사용하여 AWS 자격 증명을 설정합니다. 
   - 이 단계에서는 GitHub Secrets에 저장된 AWS 액세스 키와 비밀 키를 사용하여 AWS에 인증합니다.

6. **Deploy to S3**:
   - AWS CLI의 `s3 sync` 명령어를 사용하여 빌드된 애플리케이션을 S3 버킷에 배포합니다.
   - `--delete` 옵션을 사용하여 S3에 있는 기존 파일 중 빌드 결과에 포함되지 않은 파일을 삭제합니다. 
   - 이 단계에서 S3 버킷에 최신 애플리케이션 파일이 업로드됩니다.

7. **Invalidate CloudFront**:
   - AWS CLI의 `cloudfront create-invalidation` 명령어를 사용하여 CloudFront 캐시를 무효화합니다. 
   - 이 단계에서는 모든 경로(`/*`)에 대해 캐시를 무효화하여 사용자에게 최신 콘텐츠가 제공되도록 합니다.

## 배포 링크 및 환경

| 환경 | URL | 설명 |
|:-----|:----|:-----|
| **S3 버킷 웹사이트 엔드포인트** | http://hh-plus-chap4.s3-website.ap-northeast-2.amazonaws.com | 정적 웹 호스팅을 위한 원본 서버 역할 |
| **CloudFront 배포 도메인** | https://d3gs3udtp7lhml.cloudfront.net | CDN을 통한 캐싱 및 전송 최적화 제공 |
| **실제 서비스 도메인** | https://weekly-app.net | Route 53을 통한 도메인 관리 (실무 환경 구성 예시) |

## 주요 개념

### GitHub Actions과 CI/CD 도구
- **GitHub Actions**는 GitHub에서 제공하는 CI/CD(지속적 통합/지속적 배포) 도구로, 코드 변경이 발생할 때마다 자동으로 빌드, 테스트, 배포 작업을 수행할 수 있게 해줍니다. 
- 이 기능 덕분에 개발자는 코드 품질을 유지하면서도 배포 프로세스를 간소화할 수 있습니다. 
- 이 프로젝트에서는 GitHub Actions를 활용해 효율적으로 배포를 관리하고 있습니다.

### S3와 스토리지
- **Amazon S3** (Simple Storage Service)는 AWS의 객체 스토리지 서비스로, 대량의 데이터를 안전하게 저장하고 관리할 수 있는 강력한 도구입니다. 
- 높은 내구성과 가용성을 제공하는 S3는 데이터 백업, 아카이빙, 웹사이트 호스팅 등 다양한 용도로 사용됩니다. 
- 이 프로젝트에서는 S3에 최신 버전의 애플리케이션을 배포하고 있습니다.

### CloudFront와 CDN
- **Amazon CloudFront**는 AWS의 콘텐츠 전송 네트워크(CDN) 서비스로, 전 세계에 분산된 엣지 로케이션을 통해 사용자에게 빠르고 안전하게 콘텐츠를 전송합니다. 
- CloudFront를 사용하면 웹사이트, API, 비디오 스트리밍 등 다양한 콘텐츠의 로딩 속도를 개선할 수 있습니다.
- 이 프로젝트에서는 S3에 배포된 애플리케이션을 CloudFront를 통해 사용자에게 더 빠르고 안정적인 경험을 제공하고 있습니다.

### 캐시 무효화(Cache Invalidation)
- **캐시 무효화**는 CDN에서 캐시된 콘텐츠를 업데이트하거나 삭제하는 과정입니다. 
- 사용자가 콘텐츠를 변경하거나 새로운 버전을 배포할 때, 이전 버전의 캐시가 남아있으면 사용자에게 오래된 정보가 제공될 수 있습니다. 
- 이 프로젝트에서는 main 브랜치에 병합될 때 모든 경로(`/*`)에 대해 캐시를 무효화하여 항상 최신 콘텐츠를 사용자에게 제공하고, 웹사이트의 신뢰성을 유지하고 있습니다.

## Route 53과 DNS
- **Amazon Route 53**은 AWS의 DNS 서비스로, 웹사이트와 클라우드 리소스에 대한 도메인 이름을 관리하고 사용자 요청을 최적의 엔드포인트로 라우팅합니다.
- 도메인 이름을 IP 주소로 변환하는 DNS 레코드를 생성하고 관리합니다.
- Route 53을 통해 도메인을 직접 등록하고 관리할 수 있습니다. (https://weekly-app.net/ 로 접속 시 Route 53가 도메인을 라우팅합니다.)
- 이 프로젝트에서는 Route 53을 통해 사용자 요청을 최적의 엔드포인트로 라우팅하고 있습니다.

### Repository secret과 환경변수
- **Repository secret**은 GitHub 저장소에서 민감한 정보를 안전하게 저장하고 관리하는 방법입니다. 
- 이 정보는 key-value 형태로 저장되며 외부에 공개되지 않기 때문에, AWS 키와 같은 보안이 필요한 정보들을 안전하게 보관할 수 있습니다. 
- 이 프로젝트에서는 AWS 키와 같은 중요한 정보를 Repository secret을 통해 관리하여, 보안을 강화하고 있습니다. 
- 현재 사용 중인 환경 변수:
  - AWS_ACCESS_KEY_ID: IAM 계정 생성시 발급받은 액세스 키
  - AWS_SECRET_ACCESS_KEY: IAM 계정 생성시 발급받은 비밀 액세스 키
  - AWS_REGION: S3를 세팅한 리전(지역)의 코드
  - AWS_PRODUCTION_BUCKET_NAME: 빌드 산출물을 업로드할 S3 버킷 이름
  - DISTRIBUTION: CloudFront 배포 ID
  - SKIP_PREFLIGHT_CHECK: 프리플라이트 체크 스킵 옵션
## S3 vs CloudFront 비교

| *S3 직접 접근 시 측정 결과 1* | *CloudFront 접근 시 측정 결과 1*|
|:---:|:---:|
| <img src="https://github.com/user-attachments/assets/84f3d3a5-16f7-4a06-a5aa-6df0d326d0f5" width="400" height="200" alt="S3 접근 성능 1"> | <img src="https://github.com/user-attachments/assets/772c495a-c759-4f89-b0d6-4b645d277506" width="400" height="200" alt="CloudFront 성능 1"> |
| *S3 직접 접근 시 측정 결과 2* | *CloudFront 접근 시 측정 결과 2* |
| <img src="https://github.com/user-attachments/assets/24c65632-8f12-47c7-9f56-caf95e7cf387" width="400" height="200" alt="S3 접근 성능 2"> | <img src="https://github.com/user-attachments/assets/b8080309-9798-4a53-bd4c-8c4ba221b6af" width="400" height="200" alt="CloudFront 성능 2"> |
| *S3 (DOM, FCP, LCP)* | *CloudFront (DOM, FCP, LCP)* |
| <img src="https://github.com/user-attachments/assets/ddc9707d-5494-452e-a138-85018bbb1426" width="400" height="200" alt="S3 (DOM, FCP, LCP)"> | <img src="https://github.com/user-attachments/assets/d1cf7874-5e3e-4a6b-aff8-032d765071ff" width="400" height="200" alt="CloudFront (DOM, FCP, LCP)"> |

- CloudFront에서만 Content Encoding 기능이 적용되어 있습니다. 
  - **Content-Encoding: br**: 이 헤더는 Brotli 압축 알고리즘을 사용하여 콘텐츠가 압축되었음을 나타냅니다.
  - 이 기능은 웹 콘텐츠를 압축하여 전송함으로써 데이터 전송량을 줄이고, 페이지 로딩 속도를 개선하는 데 도움을 줍니다. 
- **X-Cache: Hit from CloudFront**: 이 헤더는 요청된 콘텐츠가 CloudFront의 캐시에서 제공되었음을 나타냅니다. 
  - **캐시 히트**: 캐시에서 제공된 경우, 응답 속도가 빨라지고 원본 서버의 부하가 줄어듭니다. 
    - 사용자에게 더 나은 성능을 제공하며, 대역폭 사용량을 절감하는 데 기여합니다.
  - **캐시 미스**: 반대로, 만약 `X-Cache` 헤더가 `Miss from CloudFront`로 표시된다면, 요청된 리소스가 캐시에 없어서 원본 서버에서 직접 가져왔음을 의미합니다. 
    - 이 경우 응답 속도가 느려질 수 있습니다.

### CDN 성능 개선 보고서
- Chrome DevTools Network 탭 활용
- Performance Insights 지표 측정
#### Network 탭 성능

| 성능 지표            | S3 직접 접근 | CloudFront 접근 | 개선 비율 (%) |
|:-------------------|:------------:|:---------------:|:-------------:|
| **Load Time**          | 681ms    | 186ms       | `72.6%`     |
| **DOMContentLoaded (network)**    | 451ms    | 48ms        | `89.4%`     |
| **Finish**             | 729ms    | 318ms       | `56.4%`     |

---

#### Performance Insight

| 성능 지표            | S3 직접 접근 | CloudFront 접근 | 개선 비율 (%) |
|:-------------------|:------------:|:---------------:|:-------------:|
| **DOMContentLoaded (insight)** | 140ms    | 50ms       | `64.3%`     |
| **First Contentful Paint (FCP)** | 200ms    | 80ms       | `60.0%`     |
| **Largest Contentful Paint (LCP)** | 200ms    | 90ms       | `55.0%`     |


<small>
 *Load Time: 사용자가 페이지를 요청한 시점부터 모든 리소스가 완전히 로드되어 사용자에게 표시될 때까지 걸린 총 시간입니다.<br/>
 *DOMContentLoaded: HTML 문서가 완전히 로드되고 파싱이 완료된 시점입니다.<br/>
 *Finish: 모든 요청이 완료된 시점입니다.<br/>
 *FCP: 첫 번째 콘텐츠가 화면에 표시된 시점입니다.<br/>
 *LCP: 가장 큰 콘텐츠가 화면에 표시된 시점입니다.<br/>
</small>

#### 성능 최적화 요소
1) **콘텐츠 압축**
   - CloudFront의 Brotli 압축으로 전송 데이터 크기 감소
   - Content-Encoding: br 헤더 적용

2) **캐시 활용**
   - X-Cache: Hit from CloudFront
   - 엣지 로케이션에서의 즉시 응답

3) **글로벌 배포**
   - 전세계 엣지 로케이션 활용
   - 사용자 위치 기반 최적 경로 제공

## 결론
- Amazon S3에서 CloudFront로의 전환은 웹 애플리케이션의 성능을 크게 향상시켰습니다. 
- 로딩 시간, DOMContentLoaded, FCP, LCP 등 주요 성능 지표에서 CloudFront가 S3에 비해 55%에서 89%까지 개선된 결과를 보였습니다.
- 특히, CloudFront의 캐싱 기능과 콘텐츠 압축은 사용자에게 더 빠르고 안정적인 경험을 제공하며, 서버 부하를 줄이는 데 기여했습니다. 
- 이러한 성능 개선은 사용자 만족도를 높이고, 웹사이트의 신뢰성을 강화하는 데 중요한 역할을 합니다.
- 결론적으로, CloudFront를 통한 CDN 활용은 웹 애플리케이션의 성능 최적화에 효과적이며, 향후 더 많은 사용자에게 안정적인 서비스를 제공할 수 있는 기반이 됩니다.

---
<small>
*참고: 성능 측정은 Chrome DevTools의 Network 탭을 사용하여 측정되었습니다.
</small>