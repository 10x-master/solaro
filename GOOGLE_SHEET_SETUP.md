# 솔라로(주) 모바일 랜딩페이지 - 구글 시트 CRM 실시간 연동 가이드

`index.html` 파일 내 상담 신청 폼(`crmConsultForm`)은 무료 구글 시트(Google Sheets) 및 구글 앱스 스크립트(Google Apps Script)와 100% 무료로 완벽 연동됩니다.

---

### 1단계: 구글 스프레드시트 생성
1. [구글 스프레드시트](https://sheets.new)에 접속하여 새 시트를 생성합니다.
2. 시트 이름을 `솔라로_상담신청DB` 등으로 지정합니다.
3. 1행(헤더)에 다음 열 이름을 입력합니다:
   - `A1`: **접수일시**
   - `B1`: **연락처**
   - `C1`: **성함**
   - `D1`: **회사/기관명**
   - `E1`: **문의내용**
   - `F1`: **유입경로**

---

### 2단계: Apps Script 코드 복사 & 붙여넣기
1. 상단 메뉴에서 **[확장 프로그램] > [Apps Script]**를 클릭합니다.
2. 기본 코드를 모두 지우고 아래 코드를 그대로 붙여넣습니다:

```javascript
function doPost(e) {
  try {
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    var data = JSON.parse(e.postData.contents);
    
    // 한국 시간(KST)으로 타임스탬프 포맷
    var formattedDate = Utilities.formatDate(new Date(), "Asia/Seoul", "yyyy-MM-dd HH:mm:ss");
    
    // 시트에 새 행 추가
    sheet.appendRow([
      formattedDate,
      data.phone || '',
      data.name || '',
      data.company || '',
      data.message || '',
      data.referrer || ''
    ]);
    
    return ContentService.createTextOutput(JSON.stringify({
      result: "success",
      message: "접수 완료"
    })).setMimeType(ContentService.MimeType.JSON);
    
  } catch (err) {
    return ContentService.createTextOutput(JSON.stringify({
      result: "error",
      message: err.toString()
    })).setMimeType(ContentService.MimeType.JSON);
  }
}
```

---

### 3단계: 웹 앱으로 배포하기
1. 우측 상단 파란색 **[배포] > [새 배포]**를 클릭합니다.
2. 톱니바퀴 아이콘을 눌러 **[웹 앱(Web App)]** 유형을 선택합니다.
3. 설정을 다음과 같이 지정합니다:
   - **설명**: `솔라로 상담신청 API`
   - **다음 사용자로 실행**: **나(사용자 이메일)**
   - **액세스 권한이 있는 사용자**: **모든 사용자(Anyone)** *(중요: 로그인 없이 폼에서 전송할 수 있어야 함)*
4. **[배포]** 버튼을 클릭하고 액세스 승인(권한 허용)을 완료합니다.
5. 배포 완료 후 나타나는 **[웹 앱 URL]**(예: `https://script.google.com/macros/s/AKfycb.../exec`)을 복사합니다.

---

### 4단계: 관리자 페이지(admin)에서 웹 앱 URL 등록 (코드 수정 불필요!)
1. 웹사이트 맨 하단 우측 구석의 **`admin`** 버튼을 클릭합니다.
2. 초기 비밀번호 **`1234`**를 입력하고 로그인합니다.
3. **[구글 앱스 스크립트 웹앱 URL]** 칸에 3단계에서 복사한 URL을 붙여넣습니다.
4. **[관리자 알림 이메일]**에 `cyshop.kakao.com@gmail.com`을 확인/저장합니다.
5. **[연동 테스트]** 버튼을 눌러 구글 시트에 테스트 행이 잘 들어가는지 확인합니다.
6. **[설정 저장하기]**를 누르면 모든 준비가 완료됩니다!

> **팁**: 관리자 페이지의 **[비밀번호 변경]** 탭에서 초기 비밀번호(`1234`)를 언제든지 안전한 비밀번호로 변경할 수 있습니다.

