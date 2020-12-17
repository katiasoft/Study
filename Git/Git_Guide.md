# Git - Factory 가이드

### 1. 브랜치 생성하기

```
git branch "브랜치 이름"
ex) git branch katia
```

### 2. 브랜치 접속하기

```
git checkout "브랜치 이름" # master로 돌아가고 싶을때는 "브랜치 이름"에 master 입력하기
ex) git checkout katia # 생성한 브랜치로 접속
ex) git checkout master # master로 돌아가기
```

### 3. 변경할 폴더 입력하기

```
git add. # 해당 경로 파일전체
git add "폴더명" # 해당 폴더만
ex) git add "0200_IT_인프라 1" # 이때 경로는 factory/materials에서 
```

### 4. 변경사항 입력하기

```
git commit -m "변경사항"
ex) git commit -m "0200_IT인프라 1 수정"
```

### 5. Git 서버에 올리기(master / branch)

```
git push origin master # master서버에 올리기
git push origin "브랜치명" # 브랜치서버에 올리기
ex) git push origin katia
```

### 6. 로컬 및 Git서버에서 지우기

```
git rm -rf "폴더명" # 해장 폴더를 제거하기
ex) git rm -rf "0200_IT_인프라 1" # 이때 경로는 factory/materials에서 
```

### 7. Github 홈페이지에서 Code에서 브랜치 선택하기

<img width="320" alt="스크린샷 2020-11-26 오후 5 35 41" src="https://user-images.githubusercontent.com/71860142/100327059-18485100-300e-11eb-921c-f1c87096a31f.png"> 

### 8. Github 홈페이지에서 Pull requests 선택후 master에게 합치기 요청하기

<img width="800" alt="스크린샷 2020-11-26 오후 5 37 11" src="https://user-images.githubusercontent.com/71860142/100327070-1bdbd800-300e-11eb-8f31-bf47cb14d46a.png">  

 ### ---

## (참고)

 ###  브랜치 확인하기

```
git branch # 입력시 현재 어느 브랜치에 접속되어 있는지 확인
```

### master와 브랜치 합치기(master 권한에서만 가능)

```
git merge "브랜치 명" master
git merge master "브랜치 명" # commit하기

ex)
git merge katia master
git merge master katia
```





## 



