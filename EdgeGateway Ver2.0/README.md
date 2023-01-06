# 1. 개요  
본 문서는 AAS기반 제조 데이터 수집 및 저장 솔루션 중 엣지게이트웨이 설치 방법에 대해 설명합니다.  
### ※ 2022년 04월 이후 본 문서는 엣지 게이트웨이 2.0버전 설치 방법에 대해 설명합니다.  

# 2. 솔루션 설치 환경  
엣지게이트웨이 솔루션은 *Debian 10.11.2 (buster) OS, intel CPU* 환경에서 제작되었습니다.  
설치 및 실행 테스트는 *Debian 10.11.2 , Ubuntu 18.04.06 LTS* 버전에서 진행되었습니다.  
권장 사양 및 기본 환경 구성은 다음과 같습니다.  
  
## PC 권장 사양  
* Processor : i5 이상의 멀티코어 CPU  
* Memory : 16GB DRAM  
* Disk : 256GB 이상의 SSD 디스크  
* Network : 2개 이상의 LAN 포트  
* OS : *Linux Debian 10*, *Ubuntu 18.04 LTS* 또는 기타 리눅스 기반 OS (MS Windows 미지원)  
* Python 3.6 또는 3.7 버전 설치된 환경에서 정상적으로 동작합니다.  
  
## 네트워크 정책  
본 솔루션은 별도의 네트워크 방화벽 정책 추가를 필요로 합니다. 그 정보는 다음과 같습니다.  
* **inbound 포트 허용 정책**  
> OPCUA(4840), Gateway Web(5000), tsDB Web(5001), SSH(22)  
* **Outbound 허용 정책**    
> REST http(80), OPC-UA Pubsub(1883), https(443), AMQP(5672)  
  
## 기본 환경 구성  
본 솔루션 설치에 앞서 기본적인 설정이 필요하며, 그 방법은 다음과 같습니다.  
  
### 1. 사용자 계정 생성  
아래 명령어를 이용해 사용자 계정을 생성합니다  
```admin@gateway:\~$ sudo adduser admin```  
  ※ PW : nestfield, Hostname : gateway  
  
### 2. 계정 권한 설정  
아래 명령어를 이용해 사용자의 권한을 설정합니다.  
**[1]** admin@gateway:\~$ sudo usermod –aG sudo admin  
**[2]** admin@gateway:\~$ sudo visudo 입력 후 파일 맨 아래에 다음 내용을 추가합니다.  
```admin ALL=NOPASSWD:ALL```  
![visudo](https://user-images.githubusercontent.com/82207645/114326524-27b8d880-9b70-11eb-9ec2-130a103df7d5.png)  
  
### 3. 리눅스 초기 셋업  
아래 명령어를 입력하여 호스트 이름을 설정합니다.  
```admin@gateway:\~$ sudo vi /etc/hostname 내용을 gateway로 수정```  
```admin@gateway:\~$ sudo vi /etc/hosts```  
> 아래 사진과 같이 기존 호스트 이름을 gateway로 수정합니다.  
![hostname](https://user-images.githubusercontent.com/82207645/114326517-25ef1500-9b70-11eb-87ac-14e763b451e9.png)  
  
### 4. 네트워크 셋업  
**[1]** 아래 명령어를 이용해 이더넷 인터페이스의 이름을 확인합니다.  
```admin@gateway:\~$ sudo ifconfig```  
![ifconfig](https://user-images.githubusercontent.com/82207645/114326519-2687ab80-9b70-11eb-9e44-904bd37dc3ea.png)  

**[2]** 아래 명령어 입력 후 다음의 내용을 추가합니다.  
```admin@gateway:\~$ sudo vi /etc/network/interfaces```  
```
 auto enp2s0  
 iface enp2s0 inet static  
 address xxx.xxx.xxx.xxx  
 netmask xxx.xxx.xx.xxx  
 gateway xxx.xxx.xxx.xxx  
 dns-nameservers xxx.xxx.xxx.xxx  
```  
![interfaces](https://user-images.githubusercontent.com/82207645/114326521-27204200-9b70-11eb-8221-1e599731e70b.png)  
  
### 5. 네트워크 셋업  
**[1]** 아래 명령어 중 하나를 입력하거나 시스템을 재시작합니다.  
```admin@gateway:\~$ sudo service network-manager restart```  
```admin@gateway:\~$ sudo service networking restart```  

**[2]** 아래 명령어를 입력하여 IP 주소 확인 및 핑 테스트를 실행합니다.  
* IP주소 확인  
```admin@gateway:\~$ sudo ifconfig```  
* 핑 테스트  
```admin@gateway:\~$ ping 8.8.8.8```  
![ping](https://user-images.githubusercontent.com/82207645/114326522-27b8d880-9b70-11eb-8007-88a4bc8cab6c.png)  

**[3]** 아래 명령어를 입력하여 시스템을 업데이트합니다.  
```admin@gateway:\~$ sudo apt-get update```  

### 6. 솔루션 설치 및 설정 작업  
**[1]** 아래 명령어를 차례로 입력하여, 본 레포지토리의 패키지 파일 다운로드 작업을 진행합니다.  
```cd  /home/admin```  
```curl -LO https://github.com/kosmo-nestfield/EdgeGW_Solution/raw/main/EdgeGateway%20Ver2.0/edgeInstallPackage_v2.tar```  

**[2]** 아래 명령어를 입력하여 패키지 파일의 압축을 해제합니다.  
```tar -xvf edgeInstallPackage_v2.tar```  

**[3]** 아래 명령어를 입력하여 설치 작업용 스크립트를 실행합니다.  
``` ./edgeInstallPackage_v2/install_Edge.sh ```  

**[4]** 이후 아래 명령어를 차례로 입력하여 암호 파일을 편집합니다.  
```cd sharedFolder/security```  
```./dna_encrypt admin.secured```  
```./dna_encrypt opcua.secured```  
```./dna_encrypt amqp.secured```  
```./dna_encrypt regiKey.secured```  

### 7. 엣지 게이트웨이 클라우드 등록  
**\[6. 솔루션 설치 및 설정 작업\] 까지 진행하셨으면 엣지게이트웨이 설치는 완료된 것입니다.**  
**클라우드의 설치가 완료되었다면 아래 절차를 수행하여 등록절차를 진행하시면 됩니다.**  
**[1]** 웹 브라우저를 이용하여 게이트웨이 웹 대시보드로 이동합니다.  
```http://[IP주소]:5000```  
**[2]** 우측 상단 Control 메뉴의 '등록' 버튼을 클릭합니다.  
![image](https://user-images.githubusercontent.com/82207645/161478661-582eecb4-172d-410e-9e75-a4374b73ffac.png)  
**[3]** 클라우드 서버 IP, 엣지게이트웨이 ID, 클라우드 등록 키, DB 사용 여부를 입력한 후 'Register' 버튼을 클릭합니다.  
__※ 클라우드에서 엣지게이트웨이 접속을 위한 RabbitMQ 계정 생성 시, ID와 PW를 각각 엣지게이트웨이 ID와 클라우드 등록 키로 설정해주세요.__  
![image](https://user-images.githubusercontent.com/82207645/161464501-c5ca8b7e-af73-4079-b426-dedfd9ce7e7e.png)  
- 정상 등록 시 아래와 같은 메시지가 출력됩니다.  
![image](https://user-images.githubusercontent.com/82207645/161473940-6c72ccd8-8a88-4223-9038-552e03054de9.png)  
**[4]** 이후 Data rcv rate(mps) 항목을 통해 데이터가 수집되고있음을 확인하실 수 있습니다.  
![image](https://user-images.githubusercontent.com/82207645/161475597-5a4a63f9-2c4d-43f2-937a-738ce6fbac3c.png)  
**본 절차까지 정상적으로 진행되었으면 클라우드 2D 대시보드에서도 데이터가 수집됨을 확인하실 수 있습니다.**  

### 8. 추가된 기능  
- 웹 대시보드 로그인 패스워드 변경 기능이 추가되었습니다.  
![image](https://user-images.githubusercontent.com/82207645/161477117-364cfe60-76ab-4cb2-ac12-ea92e9a371c5.png)  
- 엣지 게이트웨이 설정 및 동작 초기화 기능이 추가되었습니다.  
![image](https://user-images.githubusercontent.com/82207645/161477288-10a0761b-a565-417f-87f7-0119d40c17db.png)  
- 전체적인 데이터 카운트 숫자 초기화 기능이 추가되었습니다.(Aggregation Server의 Data Sending, Queue Full 등)  
![image](https://user-images.githubusercontent.com/82207645/161477420-f45a7d49-0313-4606-8e67-3f14eb3d496d.png)  
- copyFromOPC.sh 파일을 이용해 opcuaModule 내 syscfg.json, nodeset.xml, engineering.csv 파일을 홈 디렉토리에 복사할 수 있습니다.  
![image](https://user-images.githubusercontent.com/82207645/210957031-69780b10-76b0-4998-9773-e1345560a0bd.png)  
- copyToOPC.sh 파일을 이용해 설정 파일을 opcuaModule내 설정 파일 위치로 복사할 수 있습니다.  
![image](https://user-images.githubusercontent.com/82207645/210957272-8cd152da-5000-45bd-bfe9-1a4e6fb30ad0.png)  
- 설치 후 게이트웨이 웹 대시보드가 정상적으로 실행되지 않는다면 auxiliaryInstall.sh 실행 후 게이트웨이 웹 서비스를 재실행해주세요.  
![image](https://user-images.githubusercontent.com/82207645/210957839-91f731f9-ff0a-470e-a8e7-5fb6c7bf92a3.png)  
``` sudo systemctl restart gatewayWeb.service```  





## 솔루션 설치는 본 Repository의 다음 문서를 참고하여 진행합니다.  
* [**에지게이트웨이 교육자료.pdf**](https://github.com/kosmo-nestfield/EdgwGW_Solution/blob/main/%EC%97%90%EC%A7%80%EA%B2%8C%EC%9D%B4%ED%8A%B8%EC%9B%A8%EC%9D%B4%20%EA%B5%90%EC%9C%A1%EC%9E%90%EB%A3%8C.pdf)  
* [**0527 엣지게이트웨이 실습 매뉴얼**](https://github.com/kosmo-nestfield/EdgeGW_Solution/blob/main/%EC%97%A3%EC%A7%80%EA%B2%8C%EC%9D%B4%ED%8A%B8%EC%9B%A8%EC%9D%B4%EC%8B%A4%EC%8A%B5_5%EC%9B%9427%EC%9D%BC.pdf)  
## 설치 시 필요한 사전 정보는 아래와 같습니다.  
**설치용 gatewayHostPackage 파일명**  
* [**gatewayHostPackage_v2.tar**](https://github.com/kosmo-nestfield/EdgwGW_Solution/blob/main/EdgeGateway%20Ver2.0/gatewayHostPackage_v2.tar) : 본 Repository에 업로드되어있습니다.
  
**Docker pull용 appliation images 명**  
* control    	:	nestfield/controlmodule:220101  
* opcuamodule	:	nestfield/opcuamodule:220101  
* tsDB	    	:	machbase/machbase:6.1.15  
* broker	  	:	eclipse-mosquitto:1.6.12  
* monitor 		:	nicolargo/glances:3.1.6.1   
  
**설치 시 사용되는 최소한의 Linux 명령어**  
* ls, cd , pwd, cp, cat, nano, vi, ifconfig, tar  
* sudo apt-get update  
* sudo apt-get upgrade  
  
**원격으로 SSH를 이용하여 엣지게이트웨이에 접속하는 방법은 ["SSH 사용법"](https://github.com/kosmo-nestfield/EdgwGW_Solution/tree/main/SSH%20%EC%82%AC%EC%9A%A9%EB%B2%95) 폴더를 참고해주시기 바랍니다.**  
  
## 도커 OPCUA Module과 Control Module 이미지는 Docker Hub에 업로드되어있습니다.   
* [OPCUA Module](https://hub.docker.com/repository/docker/nestfield/opcuamodule)  
* [Control Module](https://hub.docker.com/repository/docker/nestfield/controlmodule)  
