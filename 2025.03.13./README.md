# 📌 컨텐츠 배포 자동화 - SH 파일 실행

## 📝 개요

Shell(SH) 파일을 실행하여 컨텐츠를 자동으로 배포하는 기능을 개발했다. 
SH 파일을 동적으로 읽고 실행하여 서버에 컨텐츠를 배포하는 방식이며, 이를 통해 배포 프로세스를 간소화할 수 있다.

---

## 🚀 구현 내용

### 1. 배포 API 구현 (Spring Boot)

배포 요청이 들어오면 SH 파일을 실행하는 API를 작성했다.

#### ✅ 주요 기능
1. SH 파일이 존재하는지 확인
2. `server_host`, `server_port` 등의 값을 동적으로 설정
3. SH 파일 실행 후 로그를 수집하여 결과 반환

#### ✅ 코드 예시
```java
@PostMapping("content/deploy")
public ResponseEntity<GResponse> deployContent(HttpServletResponse response, @RequestBody DeployParam param) {
    response.setHeader("Job-Log", "컨텐츠 배포 실행");
    
    String basePath = System.getProperty("user.dir") + "/deploy_scripts/";
    String shFileName = "deploy_" + param.getName() + "_" + param.getProductName() + ".sh";
    File shFile = new File(basePath, shFileName);

    if (!shFile.exists()) {
        log.error("SH 파일이 존재하지 않습니다: {}", shFile.getAbsolutePath());
        return ResponseEntity.ok(new GResponse("404", "배포에 필요한 SH 파일이 없습니다."));
    }
    
    try {
        ProcessBuilder processBuilder = new ProcessBuilder("/bin/bash", "-c",
            "server_host=\"" + param.getServerHost().replaceFirst("https?://", "") + "\" " +
            "server_port=\"" + param.getServerPort() + "\" " +
            "bash " + shFile.getAbsolutePath()
        );
        processBuilder.redirectErrorStream(true);
        
        Process process = processBuilder.start();
        BufferedReader reader = new BufferedReader(new InputStreamReader(process.getInputStream()));
        
        StringBuilder output = new StringBuilder();
        String line;
        while ((line = reader.readLine()) != null) {
            output.append(line).append("\n");
        }

        int exitCode = process.waitFor();
        if (exitCode == 0) {
            log.info("SH 파일 실행 성공: {}", shFileName);
            return ResponseEntity.ok(new GResponse("0000", "배포 성공:\n" + output));
        } else {
            log.error("SH 파일 실행 실패 (exitCode: {}): {}", exitCode, shFileName);
            return ResponseEntity.ok(new GResponse("500", "배포 실패:\n" + output));
        }
    } catch (IOException | InterruptedException e) {
        log.error("SH 파일 실행 중 오류 발생: {}", shFileName, e);
        return ResponseEntity.ok(new GResponse("500", "SH 파일 실행 중 오류 발생."));
    }
}
```

---

### 2. 클라이언트 배포 요청 처리 (JavaScript)

배포 요청을 보낼 수 있도록 JavaScript로 클라이언트 요청을 처리했다.

```javascript
$("#deploy_content").click(function () {
    let selectedRow = $table.bootstrapTable('getSelections');
    if (selectedRow.length === 0) {
        return alert('배포할 항목을 선택하세요.');
    }
    
    let params = {
        name: selectedRow[0].name,
        product_name: selectedRow[0].product_name,
        server_host: selectedRow[0].server_host,
        server_port: selectedRow[0].server_port.startsWith(":") ? selectedRow[0].server_port : ":" + selectedRow[0].server_port
    };
    
    if (confirm("배포하시겠습니까?")) {
        $.post("/api/content/deploy", params, function (response) {
            alert(response.code === '0000' ? "배포 성공" : "배포 실패: " + response.message);
        });
    }
});
```

---

### 3. Shell Script (SH 파일)

SH 파일에서 배포할 컨텐츠를 특정 서버에 전송하고 Android 및 iOS 관련 컨텐츠를 처리한다.

#### ✅ 예제 SH 파일 (`deploy_sample.sh`)
```sh
#!/bin/bash

server_host=""
server_port=""

ssh root@${server_host}${server_port} 'mkdir -p /var/www/deploy_test'

# Android 컨텐츠 배포
mkdir -p /tmp/deploy_android
cd /tmp/deploy_android || exit 1
rm -rf ./*
cp -r /path/to/source/android/* ./
zip -r hybrid_android.zip ./
scp hybrid_android.zip root@${server_host}${server_port}:/var/www/deploy_test

# iOS 컨텐츠 배포
rm -rf ./*
cp -r /path/to/source/ios/* ./
zip -r hybrid_ios.zip ./
scp hybrid_ios.zip root@${server_host}${server_port}:/var/www/deploy_test
```

---

## 📌 작업하면서 배운 점

1. SH 파일 실행 시 동적 변수 할당 가능
   - `server_host`와 `server_port` 값을 동적으로 설정하여 배포 시 서버를 유연하게 선택할 수 있음.

2. Java에서 SH 파일 실행 시 `ProcessBuilder` 사용
   - `/bin/bash -c "command"` 형태로 실행하여 여러 변수를 전달 가능.

3. 배포 로그를 수집하는 것이 중요함
   - `ProcessBuilder.redirectErrorStream(true)`를 통해 표준 출력을 통합하여 오류 로그를 효과적으로 수집.

4. 배포 자동화를 통해 생산성을 향상 가능
   - 기존 수동 배포 방식보다 훨씬 빠르고 신뢰성 있는 배포 환경을 구축할 수 있음.

---

## 향후 개선 방향
- 배포 프로세스 모니터링 기능 추가 (실시간 로그 스트리밍)
- Docker 기반 컨텐츠 배포로 전환
- 배포 롤백 기능 추가 (이전 버전으로 복구 가능하도록)

