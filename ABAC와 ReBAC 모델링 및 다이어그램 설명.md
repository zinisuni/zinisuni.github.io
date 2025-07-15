<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# ABAC와 ReBAC 모델링 및 다이어그램 설명

이전에 말씀드린 대로 **리소스 테이블에 직접 권한을 매핑하는 방식**은 RBAC의 세분화된 구현이지, ABAC나 ReBAC와는 다른 접근 방식입니다. 이제 ABAC와 ReBAC가 어떻게 다르게 모델링되고 구현되는지 상세히 설명해드리겠습니다.

## 1. ABAC (Attribute-Based Access Control) 모델링

### ABAC의 핵심 개념

ABAC는 **속성(Attributes)**을 기반으로 권한을 결정하는 동적 접근 제어 모델입니다[^1][^2]. 사용자 역할뿐만 아니라 사용자 속성, 리소스 속성, 환경 속성, 액션 속성을 조합하여 실시간으로 권한을 평가합니다.

### ABAC 시스템 아키텍처 (Mermaid)

```mermaid
graph TB
    subgraph "ABAC 시스템 아키텍처"
        User[사용자] --> PEP[Policy Enforcement Point<br/>정책 시행점]
        PEP --> PDP[Policy Decision Point<br/>정책 결정점]
        PDP --> PIP[Policy Information Point<br/>정책 정보점]
        PDP --> PAP[Policy Administration Point<br/>정책 관리점]
        PAP --> PolicyRepo[(정책 저장소)]
        PIP --> AttrStore[(속성 저장소)]
        
        subgraph "속성 카테고리"
            SubjectAttr[사용자 속성<br/>- 부서<br/>- 직책<br/>- 보안등급<br/>- 위치]
            ResourceAttr[리소스 속성<br/>- 파일 타입<br/>- 소유자<br/>- 분류등급<br/>- 생성일]
            ActionAttr[액션 속성<br/>- 읽기/쓰기<br/>- HTTP 메소드<br/>- 빈도]
            EnvironmentAttr[환경 속성<br/>- 시간<br/>- 위치<br/>- 네트워크<br/>- 디바이스]
        end
        
        PIP --> SubjectAttr
        PIP --> ResourceAttr
        PIP --> ActionAttr
        PIP --> EnvironmentAttr
    end
    
    style PEP fill:#e1f5fe
    style PDP fill:#f3e5f5
    style PIP fill:#e8f5e8
    style PAP fill:#fff3e0
```


### ABAC 정책 평가 시퀀스 다이어그램

```mermaid
sequenceDiagram
    participant User as 사용자
    participant PEP as Policy Enforcement Point
    participant PDP as Policy Decision Point
    participant PIP as Policy Information Point
    participant AttrStore as 속성 저장소
    participant PolicyRepo as 정책 저장소

    Note over User,PolicyRepo: ABAC 권한 평가 과정
    
    User->>PEP: 리소스 접근 요청
    PEP->>PDP: 권한 평가 요청<br/>(사용자, 리소스, 액션)
    
    PDP->>PIP: 사용자 속성 요청
    PIP->>AttrStore: 속성 조회<br/>(부서, 직책, 보안등급)
    AttrStore-->>PIP: 속성 값 반환
    PIP-->>PDP: 사용자 속성 반환
    
    PDP->>PIP: 리소스 속성 요청
    PIP->>AttrStore: 속성 조회<br/>(파일 타입, 소유자, 분류)
    AttrStore-->>PIP: 속성 값 반환
    PIP-->>PDP: 리소스 속성 반환
    
    PDP->>PIP: 환경 속성 요청
    PIP->>AttrStore: 속성 조회<br/>(시간, 위치, 네트워크)
    AttrStore-->>PIP: 속성 값 반환
    PIP-->>PDP: 환경 속성 반환
    
    PDP->>PolicyRepo: 적용 가능한 정책 검색
    PolicyRepo-->>PDP: 정책 규칙 반환
    
    PDP->>PDP: 정책 평가<br/>(속성 + 규칙)
    
    alt 모든 조건 만족
        PDP-->>PEP: Permit (허용)
        PEP-->>User: 접근 허용
    else 조건 불만족
        PDP-->>PEP: Deny (거부)
        PEP-->>User: 접근 거부
    end
```


### ABAC 정책 예시 (XACML 스타일)

```xml
<!-- ABAC 정책 예시: 금융 문서 접근 제어 -->
<Policy PolicyId="FinancialDocumentPolicy">
    <Rule RuleId="FinancialAccess" Effect="Permit">
        <Target>
            <AnyOf>
                <AllOf>
                    <Match MatchId="string-equal">
                        <AttributeValue DataType="string">financial_document</AttributeValue>
                        <AttributeDesignator AttributeId="resource.type"/>
                    </Match>
                </AllOf>
            </AnyOf>
        </Target>
        <Condition>
            <Apply FunctionId="and">
                <!-- 사용자 속성 조건 -->
                <Apply FunctionId="string-equal">
                    <AttributeDesignator AttributeId="subject.department"/>
                    <AttributeValue DataType="string">Finance</AttributeValue>
                </Apply>
                <!-- 환경 속성 조건 -->
                <Apply FunctionId="time-in-range">
                    <AttributeDesignator AttributeId="environment.current-time"/>
                    <AttributeValue DataType="time">09:00:00</AttributeValue>
                    <AttributeValue DataType="time">17:00:00</AttributeValue>
                </Apply>
                <!-- 네트워크 속성 조건 -->
                <Apply FunctionId="string-equal">
                    <AttributeDesignator AttributeId="environment.network"/>
                    <AttributeValue DataType="string">corporate</AttributeValue>
                </Apply>
            </Apply>
        </Condition>
    </Rule>
</Policy>
```


## 2. ReBAC (Relationship-Based Access Control) 모델링

### ReBAC의 핵심 개념

ReBAC는 **관계(Relationships)**를 기반으로 권한을 결정하는 그래프 기반 접근 제어 모델입니다[^3][^4]. 사용자와 리소스, 그리고 리소스 간의 관계를 통해 권한을 동적으로 계산합니다.

### ReBAC 그래프 구조 다이어그램

```mermaid
graph TB
    subgraph "ReBAC 관계 그래프 모델"
        subgraph "사용자 계층"
            Alice[Alice<br/>사용자]
            Bob[Bob<br/>사용자]
            Charlie[Charlie<br/>사용자]
        end
        
        subgraph "조직 계층"
            EngineeringTeam[Engineering Team<br/>팀]
            MarketingTeam[Marketing Team<br/>팀]
            Company[Company<br/>조직]
        end
        
        subgraph "리소스 계층"
            RootFolder[Root Folder<br/>최상위 폴더]
            ProjectFolder[Project Folder<br/>프로젝트 폴더]
            DocFolder[Docs Folder<br/>문서 폴더]
            ReadmeFile[README.md<br/>파일]
            SpecFile[spec.md<br/>파일]
        end
        
        %% 사용자-팀 관계
        Alice -.->|member| EngineeringTeam
        Bob -.->|member| EngineeringTeam
        Charlie -.->|member| MarketingTeam
        
        %% 팀-회사 관계
        EngineeringTeam -.->|part_of| Company
        MarketingTeam -.->|part_of| Company
        
        %% 사용자-리소스 관계
        Alice -.->|owner| ProjectFolder
        Bob -.->|editor| DocFolder
        Charlie -.->|viewer| RootFolder
        
        %% 리소스 계층 관계
        RootFolder -.->|contains| ProjectFolder
        ProjectFolder -.->|contains| DocFolder
        DocFolder -.->|contains| ReadmeFile
        DocFolder -.->|contains| SpecFile
        
        %% 권한 상속 관계
        ProjectFolder -.->|inherits_from| RootFolder
        DocFolder -.->|inherits_from| ProjectFolder
        ReadmeFile -.->|inherits_from| DocFolder
        SpecFile -.->|inherits_from| DocFolder
    end
    
    style Alice fill:#e3f2fd
    style Bob fill:#e3f2fd
    style Charlie fill:#e3f2fd
    style EngineeringTeam fill:#f3e5f5
    style MarketingTeam fill:#f3e5f5
    style Company fill:#f3e5f5
    style RootFolder fill:#e8f5e8
    style ProjectFolder fill:#e8f5e8
    style DocFolder fill:#e8f5e8
    style ReadmeFile fill:#fff3e0
    style SpecFile fill:#fff3e0
```


### Google Zanzibar 스타일 ReBAC 아키텍처

```mermaid
graph TB
    subgraph "Zanzibar 스타일 ReBAC 시스템"
        subgraph "클라이언트 계층"
            App1[Application 1]
            App2[Application 2]
            App3[Application 3]
        end
        
        subgraph "Zanzibar 서비스 계층"
            ZanzibarAPI[Zanzibar API]
            
            subgraph "핵심 컴포넌트"
                Tuple[Tuple Store<br/>관계 저장소]
                Eval[Evaluation Engine<br/>평가 엔진]
                Schema[Schema Store<br/>스키마 저장소]
            end
            
            subgraph "API 메소드"
                Read[Read<br/>관계 조회]
                Write[Write<br/>관계 저장]
                Check[Check<br/>권한 확인]
                Expand[Expand<br/>사용자 확장]
                Watch[Watch<br/>변경 감지]
            end
        end
        
        subgraph "데이터 계층"
            GraphDB[(그래프 데이터베이스)]
            Cache[(캐시)]
            ChangeLog[(변경 로그)]
        end
        
        App1 --> ZanzibarAPI
        App2 --> ZanzibarAPI
        App3 --> ZanzibarAPI
        
        ZanzibarAPI --> Read
        ZanzibarAPI --> Write
        ZanzibarAPI --> Check
        ZanzibarAPI --> Expand
        ZanzibarAPI --> Watch
        
        Read --> Tuple
        Write --> Tuple
        Check --> Eval
        Expand --> Eval
        Watch --> ChangeLog
        
        Tuple --> GraphDB
        Eval --> GraphDB
        Schema --> GraphDB
        
        Eval --> Cache
        Tuple --> Cache
        
        Write --> ChangeLog
    end
    
    style ZanzibarAPI fill:#e1f5fe
    style Tuple fill:#f3e5f5
    style Eval fill:#e8f5e8
    style Schema fill:#fff3e0
```


### ReBAC 권한 확인 시퀀스 다이어그램

```mermaid
sequenceDiagram
    participant Client as 클라이언트 앱
    participant ZanzibarAPI as Zanzibar API
    participant Eval as 평가 엔진
    participant GraphDB as 그래프 DB
    participant Cache as 캐시

    Note over Client,Cache: ReBAC 권한 확인 과정
    
    Client->>ZanzibarAPI: check(user:alice, permission:edit, object:document:readme)
    ZanzibarAPI->>Eval: 권한 평가 요청
    
    Eval->>Cache: 캐시된 관계 확인
    alt 캐시 히트
        Cache-->>Eval: 관계 데이터 반환
    else 캐시 미스
        Eval->>GraphDB: 관계 그래프 조회
        Note over GraphDB: 1. alice → owner → project:alpha<br/>2. project:alpha → parent → document:readme<br/>3. owner + parent = edit 권한
        GraphDB-->>Eval: 관계 경로 반환
        Eval->>Cache: 결과 캐싱
    end
    
    Eval->>Eval: 관계 경로 평가<br/>(그래프 순회 + 규칙 적용)
    
    alt 유효한 관계 경로 존재
        Eval-->>ZanzibarAPI: ALLOW
        ZanzibarAPI-->>Client: 권한 허용
    else 관계 경로 없음
        Eval-->>ZanzibarAPI: DENY
        ZanzibarAPI-->>Client: 권한 거부
    end
```


## 3. ABAC vs ReBAC vs RBAC 비교

### 구조적 차이점 비교표

| 측면 | RBAC | ABAC | ReBAC |
| :-- | :-- | :-- | :-- |
| **권한 결정 기준** | 사용자 역할 | 다양한 속성 조합 | 엔티티 간 관계 |
| **데이터 구조** | 테이블 기반 | 속성-값 쌍 | 그래프 기반 |
| **정책 표현** | 역할-권한 매핑 | 조건부 규칙 (if-then) | 관계 경로 |
| **동적 평가** | 정적 | 높음 (실시간 속성) | 높음 (관계 변화) |
| **복잡도** | 낮음 | 높음 | 중간 |
| **확장성** | 제한적 | 매우 좋음 | 좋음 |

### 실제 사용 시나리오 비교

```mermaid
graph TB
    subgraph "시나리오별 모델 적용"
        subgraph "기업 문서 관리"
            RBAC1[RBAC: admin, editor, viewer 역할]
            ABAC1[ABAC: 부서 + 시간 + 보안등급 조건]
            ReBAC1[ReBAC: 소유자 → 프로젝트 → 문서 관계]
        end
        
        subgraph "소셜 미디어"
            RBAC2[RBAC: 제한적 - 복잡한 관계 표현 어려움]
            ABAC2[ABAC: 친구 수준 + 위치 + 시간 속성]
            ReBAC2[ReBAC: 친구 → 그룹 → 게시물 관계]
        end
        
        subgraph "의료 시스템"
            RBAC3[RBAC: 의사, 간호사, 환자 역할]
            ABAC3[ABAC: 전문분야 + 병원 + 응급상황]
            ReBAC3[ReBAC: 담당의 → 환자 → 기록 관계]
        end
    end
    
    style RBAC1 fill:#ffcdd2
    style RBAC2 fill:#ffcdd2
    style RBAC3 fill:#ffcdd2
    style ABAC1 fill:#c8e6c9
    style ABAC2 fill:#c8e6c9
    style ABAC3 fill:#c8e6c9
    style ReBAC1 fill:#bbdefb
    style ReBAC2 fill:#bbdefb
    style ReBAC3 fill:#bbdefb
```


## 4. 구현 복잡도 및 성능 비교

### 시스템 복잡도 분석

```mermaid
graph LR
    subgraph "구현 복잡도"
        Simple[간단함] --> Medium[중간] --> Complex[복잡함]
        
        RBAC["RBAC
        테이블 조인
        정적 매핑"] --> Simple
        ReBAC["ReBAC
        그래프 순회
        관계 계산"] --> Medium
        ABAC["ABAC
        다중 속성 평가
        동적 조건 처리"] --> Complex
    end
    
    subgraph "성능 특성"
        Fast[빠름] --> Moderate[보통] --> Slow[느림]
        
        RBAC2["RBAC
        O(1) 룩업"] --> Fast
        ReBAC2["ReBAC
        O(depth) 순회"] --> Moderate
        ABAC2["ABAC
        O(n) 속성 평가"] --> Slow
    end
    
    style RBAC fill:#c8e6c9
    style ReBAC fill:#bbdefb
    style ABAC fill:#fff3e0
    style RBAC2 fill:#c8e6c9
    style ReBAC2 fill:#bbdefb
    style ABAC2 fill:#fff3e0
```


## 5. 하이브리드 접근법

실제 현대 시스템에서는 이 세 모델을 조합하여 사용하는 것이 일반적입니다[^5][^6]:

```mermaid
graph TB
    subgraph "하이브리드 권한 관리 시스템"
        subgraph "기본 권한 계층"
            RBAC[RBAC 기본 역할<br/>- admin, manager, user]
        end
        
        subgraph "세밀한 권한 제어"
            ReBAC[ReBAC 관계 기반<br/>- 소유자, 멤버, 뷰어]
            ABAC[ABAC 속성 기반<br/>- 시간, 위치, 디바이스]
        end
        
        subgraph "통합 정책 엔진"
            PolicyEngine[정책 엔진]
            Decision[최종 권한 결정]
        end
        
        RBAC --> PolicyEngine
        ReBAC --> PolicyEngine
        ABAC --> PolicyEngine
        PolicyEngine --> Decision
    end
    
    style RBAC fill:#c8e6c9
    style ReBAC fill:#bbdefb
    style ABAC fill:#fff3e0
    style PolicyEngine fill:#f3e5f5
```


## 결론

**RBAC**는 단순하고 관리하기 쉽지만 복잡한 권한 요구사항에는 한계가 있습니다. **ABAC**는 매우 유연하고 세밀한 제어가 가능하지만 구현과 관리가 복잡합니다[^1][^2]. **ReBAC**는 자연스러운 관계 모델링이 가능하고 확장성이 좋지만 그래프 기반 시스템의 복잡성을 갖습니다[^3][^4].

현대의 복잡한 애플리케이션에서는 이 세 모델을 상황에 맞게 조합하여 사용하는 것이 가장 효과적입니다. 예를 들어, 기본 역할 구조는 RBAC로, 리소스 간의 계층적 관계는 ReBAC로, 동적 조건들은 ABAC로 처리하는 하이브리드 접근법이 널리 사용되고 있습니다.

<div style="text-align: center">⁂</div>

[^1]: https://en.wikipedia.org/wiki/Attribute-based_access_control

[^2]: https://www.crowdstrike.com/en-us/cybersecurity-101/identity-protection/attribute-based-access-control-abac/

[^3]: https://workos.com/guide/what-is-rebac

[^4]: https://permify.co/post/relationship-based-access-control-rebac/

[^5]: https://www.aserto.com/blog/abac-vs-rebac-fine-grained-access-control

[^6]: https://www.permit.io/blog/abac-vs-rebac

[^7]: https://www.citrix.com/blogs/2022/05/17/abac-vs-rbac-comparison/

[^8]: https://en.wikipedia.org/wiki/Relationship-based_access_control

[^9]: https://www.onelogin.com/learn/rbac-vs-abac

[^10]: https://shiftasia.com/community/attribute-based-access-control-system/

[^11]: https://www.splunk.com/en_us/blog/learn/rbac-vs-abac.html

[^12]: https://www.okta.com/blog/2020/09/attribute-based-access-control-abac/

[^13]: https://aws.amazon.com/blogs/database/graph-powered-authorization-relationship-based-access-control-for-access-management/

[^14]: https://www.aserto.com/blog/rbac-vs-abac-authorization-models

[^15]: https://www.sailpoint.com/identity-library/what-is-attribute-based-access-control

[^16]: https://aws.amazon.com/blogs/security/how-to-implement-relationship-based-access-control-with-amazon-verified-permissions-and-amazon-neptune/

[^17]: https://www.okta.com/identity-101/role-based-access-control-vs-attribute-based-access-control/

[^18]: https://www.permit.io/blog/what-is-abac

[^19]: https://www.permit.io/blog/what-is-rebac

[^20]: https://www.syteca.com/en/blog/rbac-vs-abac

[^21]: https://nvlpubs.nist.gov/nistpubs/specialpublications/nist.sp.800-162.pdf

[^22]: https://www.osohq.com/academy/relationship-based-access-control-rebac

[^23]: https://py-abac.readthedocs.io/en/latest/concepts.html

[^24]: https://www.geeksforgeeks.org/system-design/attribute-based-access-controlabac/

[^25]: https://blog.netwrix.com/attribute-based-access-control-abac

[^26]: https://www.nccoe.nist.gov/sites/default/files/legacy-files/abac-nist-sp1800-3-draft.pdf

[^27]: https://www.nextlabs.com/products/application-enforcer/abac/

[^28]: https://stytch.com/blog/what-is-abac/

[^29]: https://www.ubicloud.com/docs/architecture/attribute-based-access-control-abac

[^30]: https://axiomatics.com/blog/intro-to-attribute-based-access-control-abac

[^31]: https://auth0.com/blog/what-is-rebac-and-how-to-implement-rails-api/

[^32]: https://workos.com/guide/google-zanzibar

[^33]: https://nat.sakimura.org/2024/08/25/summary-of-abac-vs-rebac-an-authorization-policy-showdown/

[^34]: https://authzed.com/learn/google-zanzibar

[^35]: https://www.idpartners.com.au/our-thinking/unlocking-strategic-advantage-with-graph-based-relationship-based-access-control-rebac

[^36]: https://www.permit.io/blog/what-is-google-zanzibar

[^37]: https://arxiv.org/pdf/1909.09904.pdf

[^38]: https://www.aserto.com/blog/google-zanzibar-drive-rebac-authorization-model

[^39]: https://www.aserto.com/blog/rbac-abac-and-rebac-differences-and-scenarios

[^40]: https://limm-jk.tistory.com/90

[^41]: https://www.permit.io/blog/conditions-vs-relationships-choosing-between-abac-and-rebac

[^42]: https://dl.acm.org/doi/10.1145/3029806.3029828

[^43]: https://www.styra.com/blog/rebac-to-the-future-building-policy-on-neo4j-data-with-enterprise-opa/

[^44]: https://www.bytefish.de/blog/relationship_based_acl_with_google_zanzibar.html

[^45]: https://www.freecodecamp.org/news/implement-relationship-based-access-control/

[^46]: https://research.google/pubs/zanzibar-googles-consistent-global-authorization-system/?trk=4b29643c-e00f-4ab6-ab9c-b1fb47aa1708\&sc_channel=podcast

[^47]: https://en.wikipedia.org/wiki/XACML

[^48]: https://docs.permit.io/how-to/build-policies/rebac/overview/

[^49]: https://heimdalsecurity.com/blog/the-complete-guide-to-xacml/

[^50]: https://ssojet.com/ciam-101/extended-access-control-markup-language-xacml

[^51]: https://axiomatics.com/resources/reference-library/relationship-based-access-control-rebac

[^52]: https://www.identityserver.com/documentation/enforcer/oasis/Architecture

[^53]: https://axiomatics.com/resources/reference-library/extensible-access-control-markup-language-xacml

[^54]: https://openid.net/specs/authorization-api-1_0-01.html

[^55]: https://www.techtarget.com/searchcio/definition/XACML

[^56]: https://dev.to/aws-heroes/pep-and-pdp-for-secure-authorization-with-avp-and-abac-3i1k

[^57]: https://www.permit.io/blog/relationship-based-access-control-rebac-with-open-policy-agent-opa

[^58]: https://www.nextlabs.com/blogs/what-is-a-policy-enforcement-point-pep/

[^59]: https://axiomatics.com/resources/reference-library/attribute-based-access-control-abac

[^60]: https://authzed.com/blog/exploring-rebac

