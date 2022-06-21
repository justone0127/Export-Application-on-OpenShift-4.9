## Export Application on OpenShift 4.9

개발자 콘솔의 토폴로지뷰에는 개발자 미리보기로 **Export Application** 기능이 포함되어 있습니다. 새로운 **Export Application** 기능을 OpenShift의 **Import multic doc** 옵션과 결합하여 동일한 클러스터의 다른 프로젝트 또는 다른 클러스터에서 애플리케이션을 복제할 수 있습니다. 이 기능은 현재 OperatorHub에서 제공되는 *gitops-primer Operator*를 설치하면 잠금 해제 됩니다.



### 1.GitOps-Primer Operator 설치하기

GitOps-Primer Operator는 클러스터에서 Object를 내보내고 Git 리포지토리에 저장하기 위해 Kubernetes 환경과 함께 사용 할 수 있습니다.

- OpenShift Console 접속 > Operators> OperatorHub 선택 > gitops-primer 검색 

  ![01_gitops-primer-operator](https://github.com/justone0127/Export-Application-on-OpenShift-4.9/blob/main/images/01_gitops-primer-operator.png)

- Install을 선택하여 설치를 진행합니다.

  ![02_gitops-primer-install](https://github.com/justone0127/Export-Application-on-OpenShift-4.9/blob/main/images/02_gitops-primer-install.png)

- 설치가 완료 되면 개발자 콘솔에서 **Export Application** 메뉴가 활성화된 것을 볼 수 있습니다.

  ![03_export-application](https://github.com/justone0127/Export-Application-on-OpenShift-4.9/blob/main/images/03_export-application.png)

### 2. Export Application을 통해 Object 내보내기

특정 프로젝트내의 Object를 내보내기 하려면, 개발자 콘솔의 토폴로지 뷰에서 **Export Application** 버튼을 선택합니다.

-  프로세스

  - gitops-primer job container 실행

    ![04_export_application_start](https://github.com/justone0127/Export-Application-on-OpenShift-4.9/blob/main/images/04_export_application_start.png)

  - 프로젝트의 Object들을 추출하는 작업이 Job Container로 실행 됩니다.

    ![05_gitops-primer-job-container-log](https://github.com/justone0127/Export-Application-on-OpenShift-4.9/blob/main/images/05_gitops-primer-job-container-log.png)

  - Job Container 수행이 끝나게 되면, 해당 Object들을 다운로드 받을 수 있게 웹서버 Pod가 뜨게 되고 다음과 같이 Download가 활성화 됩니다.

    ![06_gitops-primer-web-container](https://github.com/justone0127/Export-Application-on-OpenShift-4.9/blob/main/images/06_gitops-primer-web-container.png)

  - Download 활성화 

    ![07_download](https://github.com/justone0127/Export-Application-on-OpenShift-4.9/blob/main/images/07_download.png)

  - OpenShift 계정으로 로그인을 거친 후에 관련 Object들의 다운로드가 완료됩니다.

    ![08_login](https://github.com/justone0127/Export-Application-on-OpenShift-4.9/blob/main/images/08_login.png)

    ![09_download](https://github.com/justone0127/Export-Application-on-OpenShift-4.9/blob/main/images/09_download.png)



### 3. 다른 프로젝트에 배포하기

gitops-primer를 통해 export한 Application을 새로운 프로젝트를 생성한 후 배포해 보도록 하겠습니다.

배포하기 전에 **Export Application**을 통해 다운 받은 Object를 서버에 업로드 합니다.

**1) Creating a new Project**

```bash
$ oc new-project test
```

**2) 파일 압축 해제**

```bash
$ unzip demo-stage-2021-12-02T01_48_59Z.zip
```

**3) Application 배포**

```bash
$ oc apply -f ./demo-stage-2021-12-02T01_48_59Z
```

**4) Application 확인**

개발자 콘솔의 토폴로지 뷰를 통해서 배포된 애플리케이션을 확인합니다.



### 4. OpenShift ArgoCD를 이용해서 배포하기

Red Hat OpenShift GitOps는 클라우드 네이티브 애플리케이션에 대한 지속적인 배포를 구현하는 선언적인 방법입니다. Red Hat OpenShift GitOps를 사용하면 개발, 스테이징, 프로덕션과 같은 다양한 클러스터에 애플리케이션을 배포할 때 애플리케이션의 일관성을 유지할 수 있습니다. Red Hat OpenShift GitOps는 다음 작업을 자동화 하는데 도움이 됩니다.

- 클러스터의 구성, 모니터링, 스토리지 상태가 비슷한지 확인
- 알려진 상태에서 클러스터 복구또는 재 생성
- 여러 OpenShift Container Platform 클러스터에 구성 변경 사항 적용 또는 롤백
- 템플릿 구성을 다른 환경과 연결
- 스테이징에서 프로덕션까지 클러스터 전체에서 애플리케이션 배포



**4.1) GitOps Operator Install**

OpenShift 콘솔 접속 > Operators > OperatorHub > OpenShift GitOps 검색 > Install

Red Hat OpenShift GitOps Operator는 클러스터의 모든 네임스페이스에 설치 되며, 설치가 완료되면 `openshift-gitops` 네임스페이스에서 사용 할 수 있는 즉시 사용 가능한 Argo CD 인스턴스를 자동으로 설정하고 콘솔 도구 모음에 Argo CD 아이콘이 표시됩니다. 프로젝트에서 애플리케이션에 대한 후속 Argo CD 인스턴스를 생성 할수도 있습니다.



**4-2) Argo CD 인스턴스에 로그인**

`openshift-gitops` 프로젝트 선택 > 워크로드 > 시크릿  > `openshift-gitops-cluster` 시크릿 정보 확인 > 암호 복사

해당 암호를 이용하여 `admin` 계정으로 Argo CD UI에 로그인 합니다.

Argo CD UI는 콘솔 오른쪽 위의 도구 모음을 통해 접근 하거나, 네트워크 > 라우트 > `openshift-gitops-server`의 라우트 정보를 통해 접속 가능합니다.




**4-3) ArgoCD를 통한 Application 배포**

Argo CD는 Application을 만들 수 있는 대시보드를 제공합니다.

- Argo CD 대시보드에서 New App (새 앱) 버튼을 클릭하여 Argo CD Application을 추가합니다.
- 이 워크플로우의 경우 다음 구성을 사용하여 Application을 생성합니다.
  - example-nginx Application 배포
    - GENERAL
      - **Application Name** : example-nginx
      - **Project** : default
      - **SYNC POLICY** : Automatic
      - **SYNC OPTIONS** : AUTO-CREATE NAMESPACE Checked
    - SOURCE
      - **Repository URL** : https://github.com/justone0127/test-repo.git
      - **Revision** : HEAD
      - **Path** : apps
    - DESTINATION
      - **Cluster URL** : https://kubernetes.default.svc
      - **Namespace** : example-nginx
    - Directory
      - **DIRECTORY RECURSE** : Check



**4-4) Argo CD를 통해 Application을 배포 할 네임스페이스에 서비스 어카운트 추가**

Argo CD를 통해 네임스페이스에 Application을 배포하기 위해서는 서비스 어카운트 권한을 추가해 주어야 합니다. 추가 하지 않을 경우 다음과 같은 에러 메세지가 발생할 것 입니다.

- 에러 메시지 예시

  ```bash
  services is forbidden: User "system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller" cannot create resource "services" in API group "" in the namespace "example-nginx"
  ```

- 서비스 어카운트 추가

  ```bash
  $ oc adm policy add-role-to-user admin system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller -n ${NAMESPACE}
  ```

- 완료 화면
  ![10_argocd_application](https://github.com/justone0127/Export-Application-on-OpenShift-4.9/blob/main/images/10_argocd_application.png)


### 5. 참고
https://developers.redhat.com/articles/2021/10/21/whats-new-red-hat-openshift-49-console#developer_console_features_for_openshift_pipelines
https://github.com/konveyor/gitops-primer
  
