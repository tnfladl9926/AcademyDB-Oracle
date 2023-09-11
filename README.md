# OracleDB Project - Center Management System
Oracle DB와 sql developer를 기반으로 취업지원 학원의 전산시스템을 설계하고, 요구사항을 충족하는  ANSI-SQL, PL/SQL문을 작성하였습니다.  
<br/>


## 🖥 프로젝트 소개
취업지원 학원 전산시스템의 ERD를 설계한 후 관리자, 교사, 교육생의 요구사항을 충족하는 다양한 ANSI-SQL, PL/SQL문을 작성하였습니다.  
<br/>

## 📅 개발기간
* 23.03.27 ~ 23.04.07  
<br/>

## :two_men_holding_hands: 팀원 구성
 - 김수림: 시험관리, 성적관리, 취업현황 관리(관리자 요구사항), 시험 관리(교사 요구사항), 팀 조회(교육생 요구사항)
 - 양진영: 기초정보 관리, 개설과목 관리, 계정관리(관리자 요구사항), 시험배점 관리, 자료실 관리(교사 요구사항)
 - 장기성: 교육생 관리, 교육생 출결관리(관리자 요구사항), 근태관리, 교사 평점 조회(교사 요구사항), 교사 평점 등록, 출결관리(교육생 요구사항)
 - 이수연: 개설 과정 및 과목 스케쥴 설계, 교사 배치
 - 정상우: 개설 과정 관리, 교재 관리(관리자 요구사항), 성적 관리(교사 요구사항), 성적 조회(교육생 요구사항)
<br/>
  
## ⚙ 개발 환경
- **IDE** : SQL Developer
- **Database** : Oracle DB(11xe)  
* `EXERD`
<br/>
  
### ✔️Back-end
<img src="https://img.shields.io/badge/oracle-F80000?style=for-the-badge&logo=oracle&logoColor=white">
<br/>

## :keyboard:DB 설계
![논리 ERD](https://github.com/tnfladl9926/AcademyDB-Oracle/assets/134984241/d7231160-1e10-4c00-ba70-cb2ddfd4ba4e)
<br/>

## 📌 주요 기능

### :heavy_check_mark: 학원 시스템 설계
  - 1년에 12개의 과정이 개설되며 약 1개월 단위로 각 과정이 개설됨
  - 각 과정의 기간은 5.5개월, 6개월, 7개월으로 다름
  - 각 과정안에 5~7개의 과목이 존재하며, 각 과목은 약 1개월 단위로 교체됨
  - 과목이 교체될 때마다 해당 과목의 강의가 가능한 교사로 교체됨
  - 팀별 프로젝트 정보 조회

##### :camera: 학원 시스템 설계 이미지  
<img src="https://github.com/tnfladl9926/AcademyDB-Oracle/assets/134984241/054e2f59-6b11-48b2-87b4-fde8a6bacdab" width="50%" height="50%"/><img src="https://github.com/tnfladl9926/AcademyDB-Oracle/assets/134984241/a1facbd0-0ab3-45f1-9f19-61a406ad5a8c)" width="50%" height="50%"/>
<br/>

### :heavy_check_mark: 관리자 요구사항
  - 관리자의 교육생 취업 현황 등록 및 조회

##### :computer: PL/SQL
```
--취업현황 프로시저
create or replace procedure proc_seeGetJob(
pOpenClass number
)
is
    cursor vcursor is 
    select 
    s.s_seq as "교육생번호",
    s.name as "교육생명",
    s.tel as "전화번호",
    s.complete as "수료여부",
    gj.company as "회사명"
        from tblStudent s
            left outer join tblGetjob gj
                on s.s_seq = gj.s_Seq
                    where s.oc_seq = pOpenClass;
                    
                    vrow2 vwopenClass%rowtype;
                    
begin

select 과정명, 과정기간, 강의실명 into vrow2.과정명, vrow2.과정기간, vrow2.강의실명 from vwopensubject where 과목번호 = pOpenClass;
dbms_output.put_line('개설과정: ' || vrow2.과정명 || ' | 과정기간:' || vrow2.과정기간 || ' | 강의실명:' || vrow2.강의실명);
dbms_output.put_line('');

    for vrow in vcursor loop
        dbms_output.put_line('교육생번호: ' || vrow.교육생번호 ||' | 교육생 명:'|| vrow.교육생명 || ' | 전화번호' || vrow.전화번호 || ' | 수료여부:' ||vrow.수료여부 || ' | 회사명:' || nvl(vrow.회사명, '미취업자'));
    end loop;

exception
    when others then
        dbms_output.put_line('등록된 정보가 없습니다.');

end proc_seeGetJob;
/
```

<br/>

### :heavy_check_mark: 교사 요구사항
  - 개설 과목 별로 성적 등록 여부, 시험 문제 파일 등록 여부를 확인

##### :computer: ANSI/SQL
```
select distinct os.os_seq as "개설과목번호", 
       bs.name as "과목이름",
       ex.examtype as "시험 타입",
       ex.filename as "파일 이름",
       case when ex.examtype is null then '미등록'
            when ex.filename is null then '미등록'
            else '등록완료'
            end as "시험 등록 여부",
                case when sc.total is null then '미등록'
                else '등록완료'
                end as "성적 등록 여부"
                    from tblopensubject os
                        inner join tblbasicsubject bs
                            on os.bs_seq = bs.bs_seq
                                full outer join tblexam ex
                                    on os.os_seq = ex.os_seq
                                        full outer join tblScore sc
                                            on os.os_seq = sc.os_seq
                                                order by os.os_seq;
```
<br/>

### :heavy_check_mark: 교육생 요구사항
  - '윤선우' 교육생의 성적 정보 검색

##### :computer: ANSI/SQL
```
교육생별 성적 정보 검색
select bs.name as "개설과목",
       st.name as "교육생 이름",
       sc.pilgiScore as "필기점수",
       sc.silgiScore as "실기점수",
       sc.attendScore as "출석점수",
       sc.total as "총점"
            from tblStudent st
                inner join tblScore sc
                    on sc.s_seq = st.s_seq
                        inner join tblopenSubject os
                            on os.os_seq = sc.os_seq
                                inner join tblbasicSubject bs
                                    on bs.bs_seq = os.bs_seq
                                        where st.name = '윤선우';
```
<br/>
