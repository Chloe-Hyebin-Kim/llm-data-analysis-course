========================================================
 llm-data-analysis-course  가상환경 작업 순서 (PowerShell 기준)
========================================================

※ 프롬프트가 "PS ..."로 시작하면 PowerShell입니다.
※ 경로(D:\dev\llm-data-analysis-course)는 PC마다 다르니 그 PC의 저장소 경로로 바꿔 입력하세요.


--------------------------------------------------------
[A] 매번 작업 시작할 때 (같은 PC에서 두 번째부터)
--------------------------------------------------------

1) 프로젝트 루트로 이동
   cd D:\dev\llm-data-analysis-course

2) 가상환경 활성화
   .\.venv\Scripts\Activate.ps1

   → 프롬프트 앞에 (.venv) 가 붙으면 성공

   ※ "스크립트를 실행할 수 없으므로..." 오류가 나면:
      Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
      .\.venv\Scripts\Activate.ps1


--------------------------------------------------------
[B] 새 PC에서 처음 한 번만
--------------------------------------------------------

1) PowerShell 열기
   VS Code → 터미널 → 새 터미널  (또는 시작 메뉴에서 PowerShell)

2) 프로젝트 루트로 이동
   cd D:\dev\llm-data-analysis-course

   위치 확인:
   ls requirements.txt
   → 파일이 보이면 맞는 위치

3) 가상환경이 있는지 확인
   Test-Path .venv

   True  → 4)로
   False → 가상환경 만들기:
           python -m venv .venv

4) 가상환경 활성화
   .\.venv\Scripts\Activate.ps1
   → (.venv) 가 붙으면 성공

5) 가상환경의 Python인지 확인
   python -c "import sys; print(sys.executable)"
   → ...\llm-data-analysis-course\.venv\Scripts\python.exe 가 나와야 함

6) 패키지 설치  ← PC마다 처음 한 번만
   pip install -r requirements.txt

7) 설치 확인
   python -c "import pandas, numpy, matplotlib; print('pandas', pandas.__version__)"
   → 버전이 나오면 완료

8) VS Code 노트북 커널 선택
   노트북 오른쪽 위 [커널 선택] → .venv\Scripts\python.exe 항목 선택


--------------------------------------------------------
[C] cmd(명령 프롬프트)를 쓸 때
--------------------------------------------------------

   cd /d D:\dev\llm-data-analysis-course      ← /d 꼭 붙이기 (드라이브 이동)
   dir requirements.txt
   if not exist .venv python -m venv .venv
   .venv\Scripts\activate.bat                  ← cmd는 activate.bat
   python -c "import sys; print(sys.executable)"
   pip install -r requirements.txt

   ※ cmd에서는 # 주석을 쓰지 말 것 (명령의 일부로 인식됨)


--------------------------------------------------------
[참고] PowerShell ↔ cmd 명령 비교
--------------------------------------------------------

   하려는 일          PowerShell                     cmd
   ---------------    ----------------------------   ---------------------------
   폴더 이동          cd D:\경로                     cd /d D:\경로
   가상환경 활성화    .\.venv\Scripts\Activate.ps1   .venv\Scripts\activate.bat
   환경 변수 설정     $env:PYTHONPATH="."            set PYTHONPATH=.
   이동 후 돌아오기   Push-Location / Pop-Location   pushd / popd