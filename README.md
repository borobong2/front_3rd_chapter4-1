# S3와 CloudFront 성능 비교 분석

이 문서는 Amazon S3와 CloudFront CDN을 사용했을 때의 성능 차이를 비교 분석합니다.

## 다이어그램
<img src="https://github.com/user-attachments/assets/24522b4e-0ac0-4785-a698-0444bc3a48e1" width="800" height="400" alt="다이어그램">

## 배포 워크플로우 설명

이 GitHub Actions 워크플로우는 코드 변경 사항이 `main` 브랜치에 대한 Pull Request가 닫힐 때 자동으로 실행되며, 수동으로도 트리거할 수 있습니다. 이 워크플로우는 Node.js 애플리케이션을 빌드하고, AWS S3에 배포하며, CloudFront 캐시를 무효화하는 과정을 포함합니다.

### 주요 단계

1. **Checkout repository**:
   - `actions/checkout@v2`를 사용하여 현재 저장소의 코드를 체크아웃합니다. 이 단계는 이후 단계에서 코드에 접근할 수 있도록 합니다.

2. **Setup Node.js**:
   - `actions/setup-node@v2`를 사용하여 Node.js 환경을 설정합니다. 여기서는 Node.js 버전 18을 사용하도록 지정합니다.

3. **Remove node_modules & package-lock.json**:
   - 이전에 설치된 `node_modules`와 `package-lock.json` 파일을 제거하여 깨끗한 상태에서 패키지를 설치합니다.

4. **Install packages**:
   - `npm install --legacy-peer-deps` 명령을 실행하여 필요한 Node.js 패키지를 설치합니다. `--legacy-peer-deps` 플래그는 의존성 문제를 피하기 위해 사용됩니다.

5. **Build**:
   - `unset CI` 명령을 통해 CI 환경 변수를 제거한 후, `npm run build`를 실행하여 애플리케이션을 빌드합니다. 이 단계에서 애플리케이션의 최종 결과물이 생성됩니다.

6. **Configure AWS credentials**:
   - `aws-actions/configure-aws-credentials@v1`를 사용하여 AWS 자격 증명을 설정합니다. 이 단계에서는 GitHub Secrets에 저장된 AWS 액세스 키와 비밀 키를 사용하여 AWS에 인증합니다.

7. **Deploy to S3**:
   - `jakejarvis/s3-sync-action@master`를 사용하여 빌드된 애플리케이션을 S3 버킷에 배포합니다. `--delete` 옵션을 사용하여 S3에 있는 기존 파일 중 빌드 결과에 포함되지 않은 파일을 삭제합니다. 이 단계에서 S3 버킷에 최신 애플리케이션 파일이 업로드됩니다.

8. **Invalidate CloudFront**:
   - `chetan/invalidate-cloudfront-action@v2`를 사용하여 CloudFront 캐시를 무효화합니다. 이 단계에서는 모든 경로(`/*`)에 대해 캐시를 무효화하여 사용자에게 최신 콘텐츠가 제공되도록 합니다.

## 배포 링크
- S3 링크: [http://hh-plus-chap4.s3-website.ap-northeast-2.amazonaws.com](http://hh-plus-chap4.s3-website.ap-northeast-2.amazonaws.com/)
- CloudFront 링크: [https://d3gs3udtp7lhml.cloudfront.net](https://d3gs3udtp7lhml.cloudfront.net/)
- 실제 배포 링크: [https://weekly-app.net/](https://weekly-app.net/)

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
- 이 프로젝트에서는 캐시 무효화를 통해 항상 최신 콘텐츠를 사용자에게 제공하고, 웹사이트의 신뢰성을 유지하고 있습니다.

## Route 53과 DNS
- **Amazon Route 53**은 AWS의 DNS 서비스로, 웹사이트와 클라우드 리소스에 대한 도메인 이름을 관리하고 사용자 요청을 최적의 엔드포인트로 라우팅합니다.
- 도메인 이름을 IP 주소로 변환하는 DNS 레코드를 생성하고 관리합니다.
- Route 53을 통해 도메인을 직접 등록하고 관리할 수 있습니다. (https://weekly-app.net/ 로 접속 시 Route 53가 도메인을 라우팅합니다.)
- 이 프로젝트에서는 Route 53을 통해 사용자 요청을 최적의 엔드포인트로 라우팅하고 있습니다.

### Repository secret과 환경변수
- **Repository secret**은 GitHub 저장소에서 민감한 정보를 안전하게 저장하고 관리하는 방법입니다. 
- 이 정보는 key-value 형태로 저장되며 외부에 공개되지 않기 때문에, AWS 키와 같은 보안이 필요한 정보들을 안전하게 보관할 수 있습니다. 
- 이 프로젝트에서는 AWS 키와 같은 중요한 정보를 Repository secret을 통해 관리하여, 보안을 강화하고 있습니다. 

## S3 vs CloudFront 비교

| *S3 직접 접근 시 측정 결과 1* | *CloudFront 접근 시 측정 결과 1*|
|:---:|:---:|
| <img src="https://github.com/user-attachments/assets/84f3d3a5-16f7-4a06-a5aa-6df0d326d0f5" width="400" height="200" alt="S3 접근 성능 1"> | <img src="https://github.com/user-attachments/assets/772c495a-c759-4f89-b0d6-4b645d277506" width="400" height="200" alt="CloudFront 성능 1"> |
| *S3 직접 접근 시 측정 결과 2* | *CloudFront 접근 시 측정 결과 2* |
| <img src="https://github.com/user-attachments/assets/24c65632-8f12-47c7-9f56-caf95e7cf387" width="400" height="200" alt="S3 접근 성능 2"> | <img src="https://github.com/user-attachments/assets/b8080309-9798-4a53-bd4c-8c4ba221b6af" width="400" height="200" alt="CloudFront 성능 2"> |

### 성능 비교 결과

| 성능 지표            | S3 직접 접근 | CloudFront 접근 |
|:-------------------|:------------:|:---------------:|
| Load Time          | **681ms**    | **186ms**       |
| DOMContentLoaded    | **451ms**    | **48ms**        |
| Finish             | **729ms**    | **318ms**       |
- CloudFront 접근 시 성능이 더 빠른 이유는 캐시 무효화 때문입니다.
- CloudFront에서만 Content Encoding 기능이 적용되어 있습니다. 
  - 이 기능은 웹 콘텐츠를 압축하여 전송함으로써 데이터 전송량을 줄이고, 페이지 로딩 속도를 개선하는 데 도움을 줍니다. 
- **X-Cache: Hit from CloudFront**: 이 헤더는 요청된 콘텐츠가 CloudFront의 캐시에서 제공되었음을 나타냅니다. 
  - **캐시 히트**: 캐시에서 제공된 경우, 응답 속도가 빨라지고 원본 서버의 부하가 줄어듭니다. 
    - 사용자에게 더 나은 성능을 제공하며, 대역폭 사용량을 절감하는 데 기여합니다.
  - **캐시 미스**: 반대로, 만약 `X-Cache` 헤더가 `Miss from CloudFront`로 표시된다면, 요청된 리소스가 캐시에 없어서 원본 서버에서 직접 가져왔음을 의미합니다. 
    - 이 경우 응답 속도가 느려질 수 있습니다.


<small>
 *Load Time: 사용자가 페이지를 요청한 시점부터 모든 리소스가 완전히 로드되어 사용자에게 표시될 때까지 걸린 총 시간입니다.<br/>
 *DOMContentLoaded: HTML 문서가 완전히 로드되고 파싱이 완료된 시점입니다.<br/>
 *Finish: 모든 요청이 완료된 시점입니다.<br/>
 **Content-Encoding: br**: 이 헤더는 Brotli 압축 알고리즘을 사용하여 콘텐츠가 압축되었음을 나타냅니다.
</small>

### CloudFront 주요 이점
1. 글로벌 엣지 로케이션을 통한 빠른 컨텐츠 전송
2. HTTPS 지원으로 보안 강화
3. 캐싱을 통한 원본 서버 부하 감소
4. DDoS 보호 기능

---
<small>
*참고: 성능 측정은 Chrome DevTools의 Network 탭을 사용하여 측정되었습니다.*
</small>