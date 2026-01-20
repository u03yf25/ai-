# python-
平时练习python区，可以进行井字棋
[50105958.py](https://github.com/user-attachments/files/24738258/50105958.py)
# Q1
def stringToNum(string):
   
    number=string.split(',')
    num=list(map(int,number))
    return num
if __name__ == "__main__":
    print(stringToNum("5,6,7,8")) 
 # split(',') 的作用就是 “按逗号切分字符串”
# if __name__ == "__main__" 就是给代码加个「使用场景过滤器」：
# 自己单独用（直接运行）：执行测试 / 主逻辑；
# 给别人当工具用（导入）：只给函数 / 功能，不执行多余代码

# Q2
def _w_h_input(text,num1,num2):
    text_upper=text.upper()
    num1_round=round(num1,1)
    num2_round=round(num2,1)
    resluttuple=(text_upper,num1_round,num2_round)
    return resluttuple
# Q3
def n_w_h_output(name,weight,height):
    print(f"{name}’s weight is {weight} and his/her height is {height}")

# Q4
def calcBMI(weight,height):
    bmi_value = weight / height / height * 10000
    bmi_rounded = round(bmi_value, 1)
    return bmi_rounded

# Q5
def  bmiCat(number):
    if number<=18.5:
        print('Lower than 18.5, Underweight')
    elif 18.5<=number<=24.9:
        print('Between 18.5 and 24.9, Healthy')
    elif 25<=number<=29.9:
        print('Between 25 and 29.9, Overweight')
    elif 30<=number<=39.9:
        print('Between 30 and 39.9, Obese')
    else:
        print('More than 40, Severely obese')
    
# Q6
def bmiReport(name,weight,height,bmi, weightcategory):
    inner_dict = {
        "weight": weight,
        "height": height,
        "BMI": bmi,
        "weight category": weightcategory
    }
    outer_dict = {
        name: inner_dict
    }
    return outer_dict

#Q7
def oddList(num1,num2):
    odd_number=[]
    start = min(num1, num2)
    end = max(num1, num2)
    for i in range(start,end+1):
        if i%2!=0:
            odd_number.append(i)
    return odd_number
# Q8
def reverseString(s):
# reverse() 是 列表的内置方法，仅能用于列表（list），不能直接用于字符串（str）；
   return s[::-1]

# Q9
def startAndEnd(lst):
    if lst[0]==lst[-1]:
        return True
    else:
        return False
    
# Q10
def createBoard():
# 不建议写 [["_"]*3]*3！这种写法会让 3 个子列表指向同一内存地址，修改其中一个子列表时，其他子列表会同步变化（不符合 “独立子列表” 的要求）
    row1 = ["_", "_", "_"]
    row2 = ["_", "_", "_"]
    row3 = ["_", "_", "_"]
    board = [row1, row2, row3]
    return board
# Q11
def  displayBoard(s):
    for row in s:
        print(row)
test_board = [["_","_","_"], ["_","_","_"], ["_","_","_"]]
displayBoard(test_board)
# Q12
def getMove():
    user_input=input('输入一到九数字：').strip()
    if not user_input.isdigit():
        return False 
    s=int(user_input)
    if 1<=s<=9:
        return s
    else:
        return False
# 想让用户看到结果 → 用 print；
# 想让程序后续使用结果 → 用 return；
# 函数的核心价值是返回数据，print 只是辅助展示（比如你写一个计算函数，重点是 return 结果，而不是 print 结果）；
# 一个函数可以有多个 print，但只要执行到一个 return，函数就会立刻结束。

# Q13
def intToBoard(d):
    adjust=d-1
    row=adjust//3
    line=adjust%3
    return (row,line)

# Q14
def insertToBoard(pos_tuple, board, is_x):
    row = pos_tuple[0]
    col = pos_tuple[1]
    if board[row][col] in ["X", "O"]:
        return(False,board)
    if is_x:
        board[row][col] = "X"
    else:
        board[row][col] = "O"
        return (True, board)

# Q15
def checkDraw(board):
    # 遍历棋盘的每一行
    for row in board:
        # 遍历当前行的每一个格子
        for cell in row:
            # 如果找到空白格子（"_"），直接返回False
            if cell == "_":
                return False
    # 遍历完所有格子都没找到"_"，返回True（平局）
    return True
# Q16
def checkWin(board):
    # 1. 把所有可能赢的情况列出来（8种：3行+3列+2对角线）
    # 每个小括号是一个格子的坐标（行，列），比如(0,0)就是左上角
    win_ways = [
        [(0,0), (0,1), (0,2)],  
        [(1,0), (1,1), (1,2)],  
        [(2,0), (2,1), (2,2)],  
        [(0,0), (1,0), (2,0)],  
        [(0,1), (1,1), (2,1)],  
        [(0,2), (1,2), (2,2)], 
        [(0,0), (1,1), (2,2)],  
        [(0,2), (1,1), (2,0)] 
     ]
    for way in win_ways:
        # way[0]是第一个格子的坐标，比如(0,0)，board[0][0]就是这个格子的值
        grid1 = board[way[0][0]][way[0][1]]
        grid2 = board[way[1][0]][way[1][1]]
        grid3 = board[way[2][0]][way[2][1]]
        
        if grid1 == grid2 == grid3 and grid1 != "_":
            return True  ”
    
    
        return False
