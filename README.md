# Hyper-V Vsocket Reverse Proxy

이 애플리케이션은 Vsocket을 이용해 두 VM 사이에서 TCP 리버스 프록시를 제공하는 애플리케이션입니다.
[https://github.com/gerhart01/HyperV-sockets](https://github.com/gerhart01/HyperV-sockets)에서 소스코드를 다운받아 추가 개발을 했습니다.

이 애플리케이션의 동작 구조는 다음과 같습니다.
![Application diagram2](https://raw.githubusercontent.com/Uanid/HyperV-Vsocket-Reverse-Proxy/master/.resource/capture3.PNG)

기존의 리버스 프록시는 IP프로토콜을 이용해 서버와 클라이언트를 연결하지만, 이 애플리케이션은 VSocket을 이용하기 때문에 IP프로토콜을 사용하는 애플리케이션의 간섭 없이 동작이 가능합니다.
저의 경우 물리적으로 서로 다른 네트워크 망을 가진 VM끼리 연결할 방법을 찾다가 만들게 되었습니다.


문서 개요 (Document Index)
1. 실행 요구사항 (HW/SW Requirements)
2. 사용하기 전에 (Before use)
3. 커맨드 사용 방법(Command usage)
4. 주의사항

# 실행 요구사항 (Requirements)
이 애플리케이션은 프록시 제공을 위해 프로세스간 통신 방식으로 Hyper-V Vsocket을 선택했습니다.
이에 다음과 같은 소프트웨어/하드웨어 요구사항이 있습니다.

1. Hyper-V 서비스 실행이 가능한 물리 컴퓨터
2. 프록시 클라이언트로 사용할 Windows VM 한 개 이상
3. Microsoft Visual C++ 재배포 가능 패키지
4. (선택적) 프록시 서버로 사용할 Windows 10 Host
---
1. A Real Machine that runnable Hyper-V Service
2. One or more Windows VM for use proxy client
3. Preinstalled Microsoft Visual C++ Redistributable Package
4. (Optional) Windows 10 Host for use proxy server

# 사용하기 전에 (Before use)

1. Visual C++ 재배포 패키지 설치 (Install Microsoft Visual C++ Redistributable Package)

Microsoft Visual C++ 재배포 가능 패키지를 설치해야 합니다. 이미 설치된 PC는 설치할 필요가 없습니다. (x86을 설치하면 됩니다.)

[다운로드](https://support.microsoft.com/ko-kr/help/2977003/the-latest-supported-visual-c-downloads)
[Download](https://support.microsoft.com/en-us/help/2977003/the-latest-supported-visual-c-downloads)


2. Hyper-V 애플리케이션 서비스 등록 (Register a Hyper-V application service)

Vsocket을 위한 서비스를 Hyper-V에 등록해야 합니다. 이는 TCP/IP 프로토콜에서 port와 비슷한 개념으로 생각할 수 있습니다.

아래의 스크립트를 Host에서 관리자모드(Administrator mode)인 PowerShell에서 실행하면 됩니다.
작업이 완료되면 클립보드(Ctrl + C)에 서비스 Guid가 복사되어 있습니다.

```shell script
$friendlyName = "HV Socket Demo"

# Create a new random GUID.  Add it to the services list
$service = New-Item -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices" -Name ((New-Guid).Guid)

# Set a friendly name
$service.SetValue("ElementName", $friendlyName)

# Copy GUID to clipboard for later use
$service.PSChildName | clip.exe
```
[출처](https://docs.microsoft.com/ko-kr/virtualization/hyper-v-on-windows/user-guide/make-integration-service)
[Source](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/user-guide/make-integration-service)


# 커맨드 사용 방법

#### 프록시 서버 (Proxy Server)
```
In CMD
ServerExample.exe <listen port> <listen serviceGuid>

In PowerShell
.\ServerExample.exe <listen port> '<listen serviceGuid>'

Example
.\ServerExample 3000 '{74720853-1be5-40c7-93d2-b9118f0f9006}'
```

인자 설명(parameter description):
- listen port: 리버스 프록시를 연결할 포트입니다.
- listen serviceGuid: 프록시 클라이언트가 연결하기 위한 식별용 서비스 ID입니다.

프록시 서버는 {00000000-0000-0000-0000-000000000000}으로 바인딩합니다.
즉, 모든 클라이언트에서 오는 요청을 받습니다.

#### 프록시 클라이언트 (Proxy Client)
```
In CMD
ClientExample.exe <target ip> <target port> <target serviceGuid> [<target serverGuid>]
In PowerShell
.\ClientExample.exe <target ip> <target port> '<target serviceGuid>' ['<target serverGuid>']
Example1
.\ClientExample.exe 250.164.133.112 22 '{74720853-1be5-40c7-93d2-b9118f0f9006}'
Example2
.\ClientExample.exe 250.164.133.112 22 '{74720853-1be5-40c7-93d2-b9118f0f9006}' '{a42e7cda-d03f-480c-9cc2-a4de20abb878}'
```
인자 설명(Parameter Description):
- target ip: 리버스 프록시가 연결할 목표 IP입니다. (DNS 변환 기능은 제공되지 않습니다.)
- target port: 리버스 프록시가 연결할 목표 포트입니다.
- target serviceGuid: 프록시 서버가 연결된 서비스 ID입니다.
- target serverGuid: 프록시 서버가 구동되는 VM Id입니다. (기본적으로 Host의 ID가 입력됩니다.)

# 주의사항 (Notice)
1. 어떤 이유에선지 Microsoft에서 알려주는 방법으론 Windows Server에서 프록시 애플리케이션 실행이 불가능했습니다.
실패 원인은 Vsocket 생성 실패인데, 저는 Java개발자라 자세한 이유는 모르겠습니다. 따라서 사용이 주 목적인 분들은 Host 운영체제로 Windows 10을 사용해주세요.

2. 간단하게 개발한 애플리케이션이라 Server가 Client 1개밖에 받지 못하며 프록시 연결도 1개만 지원합니다.
(다중 프록시 연결을 위해선 서버-클라이언트를 여러번 실행해야 합니다.)

3. Server는 C언어로, Client는 C++로 개발했습니다. 저도 이유는 모르겠는데 원래 예제 코드가 그렇게 있어서 그대로 따라했습니다.

4. IP프로토콜 프록시와 Vsocket 프록시의 구조 차이


![Vsocket](https://raw.githubusercontent.com/Uanid/HyperV-Vsocket-Reverse-Proxy/master/.resource/capture1.PNG)
Vsocket:
NIC를 이용하지 않기 때문에 IP를 사용하는 환경이나 애플리케이션의 영향을 받지 않습니다.

![IP](https://raw.githubusercontent.com/Uanid/HyperV-Vsocket-Reverse-Proxy/master/.resource/capture2.PNG)
IP: NIC 또는 IP환경을 같이 이용하는 애플리케이션들의 영향을 받습니다. (예: 라우팅 테이블 조작)
