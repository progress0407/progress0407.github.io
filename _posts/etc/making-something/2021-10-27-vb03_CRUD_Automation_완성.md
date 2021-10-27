---
layout: post
title: CRUD Automation 완성
subtitle: "..."
categories: etc
tags: making-something
comments: true
published: true
---

> 본 게시글은 전 블로그에서 2020년 8월 24일에 작성한 글입니다
> https://blog.naver.com/progress0407/222068926986

## CRUD 기능 개념도

![image](https://user-images.githubusercontent.com/66164361/138992706-62a4477c-83bc-487d-b021-51079428fd22.png)

![image](https://user-images.githubusercontent.com/66164361/138992795-2ba284ea-b8ca-4c02-b3a5-ad0ceffd65b9.png)

![image](https://user-images.githubusercontent.com/66164361/138992832-de01ed7d-8d5c-4e01-bd5e-6ed1f58fe1a4.png)

(농담이다..)

정말 그 어느 함수 하나라도 대충 만든 것이 없다..

가급적 클린코드에 나오는 밥아재 생각을 하면서 짜 보았다..

응집도? 결합도.. 함수는 한 일만 잘해라..

여러번 덧 씌워 만들 때마다 함수의 형태는 이뻐져 갔고,

쓸모없는 변수들은 없어져 갔다..

뿌듯하다 ㅠㅠ

```vb
'첫 인자로 무엇이 되던간에.. 받게 되는 게 문제야.
' 첫번째 인자는 되도록 Optional을 쓰지 말자
' 첫번째 인자부터 Variant로 필수로 받은뒤 Integer, String, Range로 나누어서 생각하자
Public Function hasC(src As Variant, Optional c As Variant) As Boolean

    Dim str As String

    'Integer가 아닌 Double로 return이 돼!
    If TypeName(src) = "Integer" Or TypeName(src) = "Double" Or TypeName(src) = "Double" Then
        ' 이경우 src가 row야
        str = Cells(src, c).Value

    ElseIf TypeName(src) = "String" Then
        str = src

    ElseIf TypeName(src) = "Range" Then
        str = src.Value

    Else
        has = "Missing Argument"
        hasC = has
        Exit Function

    End If

    hasC = InStr(str, "C") + InStr(str, "c") > 0

End Function
```

hasR, hasC, hasD는 모두 구조적으로 같다.

argument가 여려 형으로 입력이 되게끔 하는 것이 애먹었다..

공식적으로 overload 기능을 지원하지 않는다.

```
(src As Variant, Optional c As Variant)
```

처음에는 위와 같은 인자 받는 부분을

```
(Optional r As Integer, Optional c As Integer, src As Variant)
```

위와 같이 작성했다. 그런데 문제는 위와같이 작성할 시..

Integer에 해당하는 row, col 값 이외의 값을 받을 시 에러가 난다.

정확히는 함수 자체가 진입이 안돼는 에러인데.. ByRef

이게.. 무조건 첫값을 받게 돼 있는데..

인자의 첫값은 Integer 타입으로 할당되어있고,

넘어 올 수 있는 값은 Integer뿐만 아니라 String, Range 타입도 또한 들어 올 수 있기 때문에

문제가 되는 것이었다..

```vb
If TypeName(src) = "Integer" Or TypeName(src) = "Double" Or TypeName(src) = "Long" Then
```

정수에 대한 판단 부분이 길어진 이유는

저렇게 처리해주지 않으면 타입 에러가 뜬다..

```vb
' C, R, U, D 중 어느 하나라도 가지고 있으면 true를 반환한다.
Public Function hasCRUD(src As Variant, Optional c As Variant) As Boolean

    Dim str As String

    If TypeName(src) = "Integer" Or TypeName(src) = "Double" Or TypeName(src) = "Long" Then
        str = Cells(src, c).Value

    ElseIf TypeName(src) = "String" Then
        str = src

    ElseIf TypeName(src) = "Range" Then
        str = src.Value

    Else
        hasCRUD = "Missing Argument"
        Exit Function

    End If

    hasCRUD = hasC(str) Or hasR(str) Or hasU(str) Or hasD(str)

End Function
```

```vb
' CRUD 순서에 맞게 만들어주는 함수야
Function sortCRUD(toSortStr As String) As String
    Dim resultStr As String: resultStr = ""

    If hasC(toSortStr) Then
        resultStr = resultStr + "C"
    End If
    If hasR(toSortStr) Then
        resultStr = resultStr + "R"
    End If
    If hasU(toSortStr) Then
        resultStr = resultStr + "U"
    End If
    If hasD(toSortStr) Then
        resultStr = resultStr + "D"
    End If

    sortCRUD = resultStr

End Function
```

RCUD, DURC 등으로 정렬돼 있으면 CRUD로 바꾸어주는 함수이다.

```vb
Public Function GetColor(rng As Range, Optional return_type As Integer = 0) As Variant

    Dim colorVal As Variant

    If rng.Columns.Count <= 1 And rng.Rows.Count <= 1 Then
        colorVal = rng.Interior.Color
        GetColor = Hex(colorVal)

    Else
            GetColor = CVErr(xlErrValue)
    End If

End Function
```

Color함수이다. stackoverflow에서 보았다.

> https://stackoverflow.com/questions/24132665/return-rgb-values-from-range-interior-color-or-any-other-color-property

```vb
Public Function numToChar(num As Integer) As String
    num = num - 1
    '몫, 나머지
    Dim quotient As Integer, remainder As Integer
    If num < 26 Then
        numToChar = Chr(65 + num)
        Exit Function
    ElseIf num >= 26 Then
        'num+1을 해 주어야 52를 입력했을시 B@같은 상황이 발생하지 않아
        quotient = (num) \ 26
        remainder = (num) Mod 26
        numToChar = numToChar(quotient) & Chr(65 + remainder)
    End If

End Function
```

![image](https://user-images.githubusercontent.com/66164361/138997375-54337cd6-0cfc-4ffd-8302-0b07a481f774.png)

처음에는 2글자에 대해서만 성립했다 (num <= (26\*26) )

그러나.. 욕심이 생겨서 그간의 중등 지식을 끌어모아 재귀함수로 만들었다

이것도 애먹음

```vb
' 셀의 범위를 각 좌푯값을 반환해주는 함수야
Public Function getAreaCoord(cell As Range) As Object
'    MsgBox ("getAreaCoord 호출됨")
    ' $A$1:$B$3 처럼 반환되는 cell값에서 $를 제외한다
    Dim addr
    addr = cell.Address
    addr = Replace(addr, "$", "")

    Dim colIdx
    colIdx = InStr(addr, ":")

    Dim leftAddr, leftCol, leftRow As Integer
    Dim rightAddr, rightCol, rightRow As Integer

    ' 만일 인자로 들어 온 셀이 오로지 하나(A1)라면 A1:A1 로 만들어 준다
    If colIdx = 0 Then
        leftAddr = addr
        rightAddr = addr
'        MsgBox ("길이가 1인 경우는 지원하지 않습니다. A1:A10 형태로 입력바랍니다")
'        Exit Function
    Else
        leftAddr = Mid(addr, 1, colIdx - 1)
        rightAddr = Mid(addr, colIdx + 1, Len(addr) - colIdx)
    End If

    leftRow = Range(leftAddr).row
    leftCol = Range(leftAddr).Column

    rightRow = Range(rightAddr).row
    rightCol = Range(rightAddr).Column

    ' Pointer변수 같은 애, 하나의 Dictionary 자료형을 만들어서 참조해.
    Dim Coord As Object: Set Coord = CreateObject("Scripting.Dictionary")

    Coord.Add "leftRow", leftRow
    Coord.Add "leftCol", leftCol
    Coord.Add "rightRow", rightRow
    Coord.Add "rightCol", rightCol

'    For Each k In Coord.keys
'        MsgBox (k & " :  " & Coord.Item(k))
'        Next

    Set getAreaCoord = Coord

End Function
```

반복되는 코드가 많아서 이 함수를 만들고자 하는 것 부터가 모듈화의 첫 시작이었다.

처음엔 후회반이었지만.. 정말 모듈화하길 잘 했다.

dictionary에 대한 자료형을 가장 많이 공부한 곳이기도 하다.

key는 시작점, 끝점 행열 이름이며 (leftRow, leftCol, rightRow, rightCol)

value는 그 점의 좌푯값이다. ( 예) A1:C3 == 2, (A)1, 4, (C)3 )

```vb
Dim Coord As Object: Set Coord = CreateObject("Scripting.Dictionary")
```

이래야 dictionary형태의 특수한 object가 선언된다는 걸 몰랐다 ㅠㅠ

```vb
Set getAreaCoord = Coord
```

반환할 때는 위처럼 반환해야 한다.

​

반환하고 받아서 사용하는 게 안돼서..

tDic(), gDic() 예제 함수를 이용해서 많이 연습했다..

마지막에 반환하는 이름을 잘못 주어서.. 틀렸다..

vba 디버그 지원 환경이 너무 열악하다..

나중에는 VS Code를 이용해 보자.

```vb
' 셀값들 중에서 CRUD를 집합형태로 수집하여 리턴합니다 (예 : R, C => CR  , U RC => CRU)
Public Function maxText(cell As Range) As String
    Dim resText As String: resText = ""

    Dim Coord As Object: Set Coord = getAreaCoord(cell)
    Dim leftRow, leftCol, rightRow, rightCol As Integer

    leftRow = Coord.Item("leftRow")
    leftCol = Coord.Item("leftCol")
    rightRow = Coord.Item("rightRow")
    rightCol = Coord.Item("rightCol")

    Dim fixedRow As Integer: fixedRow = leftRow

    For c = leftCol To rightCol
        'MsgBox (TypeName(c))
        If Not hasC(resText) And hasC(fixedRow, c) Then
            resText = resText & "C"
        End If

        If Not hasR(resText) And hasR(fixedRow, c) Then
            resText = resText & "R"
        End If

        If Not hasU(resText) And hasU(fixedRow, c) Then
            resText = resText & "U"
        End If

        If Not hasD(resText) And hasD(fixedRow, c) Then
            resText = resText & "D"
        End If

        Next c

    resText = sortCRUD(resText)

    maxText = resText

End Function
```

처음에 어떻게 집합 형태의 자료형을 만들 것인지 idea에 대한 접근이 어려웠다.

```vb
' 총 테이블 갯수를 구해주는 함수
Function getTotTable(cell As Range)

    Dim Coord As Object: Set Coord = getAreaCoord(cell)
'    Dim Coord As Object: Set Coord = tDic(cell)

    Dim leftRow, leftCol, rightRow, rightCol As Integer

    leftRow = Coord.Item("leftRow")
    leftCol = Coord.Item("leftCol")
    rightRow = Coord.Item("rightRow")
    rightCol = Coord.Item("rightCol")

'    MsgBox (": " & Coord.Item("leftRow"))
'    MsgBox ("leftR: " & leftRow)

    'CRUD 중 어느하나라도 가지고 있는가
    Dim isHas

    Dim totCnt As Integer: totCnt = 0
    For r = leftRow To rightRow
        For c = leftCol To rightCol
            If hasCRUD(r, c) Then
                totCnt = totCnt + 1
            End If
            Next c
        Next r

    getTotTable = totCnt

End Function
```

가장 처음에 쓰이는 공식이다!

```vb
' 컬럼 갯수의 임의 연산
'열의 위치는 변할 수 있기 때문에, 상단의 Column의 위치는 절대경로로 설정하자
Function getColSum(cell As Range)

    Dim Coord As Object: Set Coord = getAreaCoord(cell)
    Dim leftRow, leftCol, rightRow, rightCol As Integer

    leftRow = Coord.Item("leftRow")
    leftCol = Coord.Item("leftCol")
    rightRow = Coord.Item("rightRow")
    rightCol = Coord.Item("rightCol")

    Dim colSum, r As Integer: colSum = 0
    fixedRow = leftRow
    For c = leftCol To rightCol
        If hasCRUD(fixedRow, c) Then
            임의 연산 +-*/ Cells(1, c).Value
        End If
        Next c

    임의 연산 ...

    getColSum = colSum

End Function
```

혹시 해당 산출 공식이.. 문제가 될 수 있을까 해서 저부분은 가렸다.

소숫점 절삭하는것은 Fix(실수)를 사용하면 된다.

```vb
' 색이 노랗거나 셀값이 존재하는 경우를 따로 솎아내야해
Function hasTrig(cell As Range)

    Dim Coord As Object: Set Coord = getAreaCoord(cell)
    Dim leftRow, leftCol, rightRow, rightCol As Integer

    leftRow = Coord.Item("leftRow")
    leftCol = Coord.Item("leftCol")
    rightRow = Coord.Item("rightRow")
    rightCol = Coord.Item("rightCol")

    Dim fixedRow As Integer: fixedRow = leftRow

    For c = leftCol To rightCol
        If GetColor(Cells(fixedRow, c)) = "FFFF" And hasCRUD(fixedRow, c) Then
            hasTrig = "trig"
            Exit Function
        End If
        Next c
    hasTrig = "noTrig"
End Function
```

```vb
Sub addRows()

    Application.ScreenUpdating = False

    '이 화면의 영역을 찾아야 해
    Dim leftRow, leftCol, rightRow, rightCol As Integer

    'Find Start Point : name "progNa"
    Dim flg As Boolean: flg = False
    For r = 1 To (26)
        If flg = True Then
            Exit For
        End If
        For c = 1 To (26)
            If StrComp(Cells(r, c).Value, "progNa") = 0 Then
                leftRow = r
                leftCol = c
                flg = True

                'MsgBox ("leftRow : " & leftRow)
                'MsgBox ("leftCol : " & leftCol)

                Exit For
            End If
        Next c
    Next r

    If flg = False Then
        MsgBox ("테이블 이름에 'progNa' 넣어주세요!")
        Exit Sub
    End If

    'Find End Point : 시작점 기준으로
    For r = leftRow To (26 * 26 * 26)
        If StrComp(Cells(r, leftCol).Value, "") = 0 Then
            rightRow = r - 1
            MsgBox ("rightRow : " & rightRow)
            Exit For
        End If
    Next r

    For c = leftCol To (26 * 26 * 26)
        If StrComp(Cells(leftRow, c).Value, "") = 0 Then
            rightCol = c - 1
            MsgBox ("rightCol : " & rightCol)
            Exit For
        End If
    Next c

    ' Header를 제외하고 생각하자
    leftRow = leftRow + 1

    Dim topicCRUD As Object: Set topicCRUD = CreateObject("Scripting.Dictionary")

    For r = leftRow To rightRow
        'title과 crudLIst 는 임시 저장소야. 다른 용도로 재활용 가능해
        Dim title, crudList As String
        title = Cells(r, leftCol).Value
        crudList = maxText(Range(numToChar(leftCol + 1) & r & ":" & numToChar(rightCol) & r))
        topicCRUD(title) = crudList
    Next r

    ' 변하는 Row 값
    Dim cursorRow As Integer: cursorRow = leftRow + 1
    Dim titleRow, titleCol As Integer

    For Each title In topicCRUD

        crudList = topicCRUD(title)
        titleRow = cursorRow
        titleCol = leftRow

        If hasC(crudList) Then
            Rows(cursorRow).Insert
            Cells(cursorRow, leftCol).Value = title & " 등록"
            cursorRow = cursorRow + 1
        End If

        If hasR(crudList) Then
            Rows(cursorRow).Insert
            Cells(cursorRow, leftCol).Value = title & " 조회"
            cursorRow = cursorRow + 1
        End If

        If hasU(crudList) Then
            Rows(cursorRow).Insert
            Cells(cursorRow, leftCol).Value = title & " 수정"
            cursorRow = cursorRow + 1
        End If

        If hasD(crudList) Then
            Rows(cursorRow).Insert
            Cells(cursorRow, leftCol).Value = title & " 삭제"
            cursorRow = cursorRow + 1
        End If

        cursorRow = cursorRow + 1

    Next

End Sub
```

사실 이 친구가 끝판왕이다.

앞에 있는 것을 모두 끌어 모아 사용했다.

애먹었다.

​

Function은 행/열 삽입이 되지 않는다.

그래서 프로시저, 즉 Sub형태로 작성해야하는데,

애는 또 cell An Range를 인자로 못 받아 온다..

​

그래서 먼저 내가 관심있는 Table이 어디서부터 어디까지인지를 그 영역을 조사할 판단부분을 설계해야 했다.

​

관심을 가질 Table의 좌푯값을 알아내었다면 그 다음으로

​

CRUD에 따른 행삽입이 일어나는데,

​

이 때 기존 Table에 대한 title 정보와 CRUD 중 무엇이 있는지를

추가된 행 삽입에 관계 없이 항상 알 수 있는 Dictionary가 필요하다.

​

이 가상의 Dictionary는 Key, Value를 가지는데

key는 progNa의 title이고, Value는 crudList이다.

​

> 한 바퀴 이상 for 문 탈출하기
> https://stackoverflow.com/questions/28905934/how-to-exit-more-than-1-for-loop-in-excel-vba

```vb
Function searchTwoWord(cell As Range)
    Dim str As String: str = ""
    str = cell.Value

    Dim lastTwoWord As String: lastTwoWord = Right(str, 2)

    If lastTwoWord = "등록" Then
        searchTwoWord = "EI"

    ElseIf lastTwoWord = "조회" Then
        searchTwoWord = "Select"

    ElseIf lastTwoWord = "수정" Then
        searchTwoWord = "Update"

    ElseIf lastTwoWord = "삭제" Then
        searchTwoWord = "Delete"

    Else
        searchTwoWord = ""

    End If

End Function
```

마지막 하산 작업.. 수월했다. 거의 얘만 고생 안하고 만든 듯

![image](https://user-images.githubusercontent.com/66164361/138997702-f7e1f77d-6d12-4de1-ba94-0bd95eed8f7c.png)

회사 가서 자동화 소스 옮겨 놔야 하는데.............

와...... 458 줄..............................

​
픽픽 쓰는데, 잘 안돼서

CodeSnap 이라는 Extension 사용해서 해결했다.

![image](https://user-images.githubusercontent.com/66164361/138997763-3c8e8878-ac38-44cd-b5d2-d78ddc0f2f8e.png)

![image](https://user-images.githubusercontent.com/66164361/138997790-1b10e299-80b5-4a8d-a2b4-6f8700563549.png)

짤리는 부분 조심할 것!!

width 너무 협소하면 불편하다..
