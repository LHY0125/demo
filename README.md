# 五子棋人机对战AI项目

## 项目简介
这是一个基于C语言实现的高级五子棋人机对战系统，采用α-β剪枝的极小极大算法实现AI决策，支持自定义棋盘大小和完整的游戏复盘功能。

## 核心功能
- 支持5x5到25x25的棋盘尺寸
- 玩家(X)与AI(○)对战模式
- AI采用α-β剪枝优化的极小极大算法
- 详细的棋型评估系统（活四、冲四、活三等）
- 完整的游戏复盘功能
- 清晰的棋盘界面显示

## 技术特点
1. **AI算法**：
   - 使用α-β剪枝优化的极小极大算法
   - 搜索深度可调（当前设置为3层）
   - 优先处理威胁位置（如阻止玩家形成活四）

2. **评估系统**：
   - 考虑四个方向的棋型评估
   - 区分活棋、眠棋和死棋
   - 位置奖励（中心位置价值更高）

3. **游戏流程**：
   - 完整的胜负判定
   - 平局检测
   - 详细的复盘系统

## 编译运行
1. 编译程序：
   ```bash
   gcc "C语言代码/课上代码练习/五子棋/五子棋 copy 3.c" -o gomoku -lm
   ```

2. 运行游戏：
   ```bash
   ./gomoku
   ```

## 使用说明
1. 启动后输入棋盘尺寸（5-25，默认15）
2. 游戏进行：
   - 玩家输入坐标格式：行 列（如"8 8"）
   - AI会自动计算最佳落子位置
3. 游戏结束：
   - 显示胜负结果
   - 可选择查看完整复盘

## 核心函数说明

### 1. ai_move() - AI决策函数
**功能**：决定AI的最佳落子位置  
**算法**：
1. 优先检查是否需要阻止玩家形成活四/冲四
2. 使用α-β剪枝优化的极小极大算法搜索最佳位置
3. 考虑已有棋子附近2格范围内的位置
**参数**：无  
**返回值**：无（直接修改棋盘状态）

### 2. evaluate_pos() - 棋型评估函数
**功能**：评估特定位置对当前玩家的价值  
**评分标准**：
- 活四: 100000分
- 冲四: 10000分 
- 活三: 5000分
- 眠三: 1000分
- 活二: 500分
- 眠二: 100分
**参数**：
- x,y: 待评估位置坐标
- player: 当前玩家标识
**返回值**：评估分数（越高越好）

### 3. dfs() - 深度优先搜索
**功能**：实现带α-β剪枝的极小极大算法  
**特点**：
- 搜索深度：3层
- 使用α-β剪枝优化搜索效率
- 评估函数差值作为叶节点值
**参数**：
- x,y: 当前落子位置
- player: 当前玩家
- depth: 剩余搜索深度
- alpha,beta: 剪枝参数
- is_maximizing: 是否最大化玩家

### 4. count_specific_direction() - 方向分析
**功能**：计算特定方向上的连续棋子情况  
**返回结构**：
- continuous_chess: 连续棋子数
- check_start/check_end: 两端是否开放
**应用**：用于判断棋型（活棋/眠棋）

### 5. check_win() - 胜负判断
**功能**：检查落子后是否形成五连珠  
**逻辑**：检查四个方向是否存在≥5的连续同色棋子

## 程序流程图

```
开始
│
├─> 初始化棋盘
│    │
│    └─> 设置默认大小(15x15)
│
├─> 游戏主循环
│    │
│    ├─> [玩家回合]
│    │    ├─> 显示棋盘
│    │    ├─> 获取玩家输入坐标
│    │    └─> 落子并检查胜负
│    │
│    ├─> [AI回合]
│    │    ├─> AI计算最佳落子(α-β剪枝)
│    │    └─> 落子并检查胜负
│    │
│    └─> 检查平局条件
│
├─> 游戏结束处理
│    ├─> 显示胜负结果
│    └─> 询问是否复盘
│         ├─> 是: 显示完整复盘
│         └─> 否: 退出游戏
│
└─> 程序结束
```

## 关键代码实现

### 1. AI决策核心算法
```c
// α-β剪枝极小极大算法实现
int dfs(int x, int y, int player, int depth, int alpha, int beta, int is_maximizing)
{
    if (depth == 0 || check_win(x, y))
    {
        return evaluate_pos(x, y, player) - evaluate_pos(x, y, 3 - player);
    }
    
    if (is_maximizing)
    {
        int max_eval = INT_MIN;
        for (每个可能位置)
        {
            int eval = dfs(x, y, player, depth-1, alpha, beta, 0);
            max_eval = max(max_eval, eval);
            alpha = max(alpha, eval);
            // α剪枝
            if (beta <= alpha)
            {
               break;  
            }
        }
        return max_eval;
    } 
    else
    {
        int min_eval = INT_MAX;
        for (每个可能位置)
        {
            int eval = dfs(x, y, player, depth-1, alpha, beta, 1);
            min_eval = min(min_eval, eval);
            beta = min(beta, eval);
            // β剪枝
            if (beta <= alpha)
            {
                break;
            }  
        }
        return min_eval;
    }
}
```

### 2. 棋型评估函数
```c
// 评估特定位置的棋型价值
int evaluate_pos(int x, int y, int player)
{
    int score = 0;
    for (四个方向)
    {
        ChessPattern pattern =count_specific_direction(x, y, dir);
        
        if (pattern.continuous_chess >= 5)
        {
            return 1000000; // 五连珠
        }    
        
        // 活棋
        if (pattern.check_start && pattern.check_end)
        {   
            if (pattern.continuous_chess == 4) 
                score += 100000; // 活四
            else if (pattern.continuous_chess == 3)
                score += 5000; // 活三
            else if (pattern.continuous_chess == 2)
                score += 500; // 活二
        } 
        // 眠棋
        else if (pattern.check_start || pattern.check_end)
        {   
            if (pattern.continuous_chess == 4)
                score += 10000; // 冲四
            else if (pattern.continuous_chess == 3)
                score += 1000; // 眠三
            else if (pattern.continuous_chess == 2)
                score += 100; // 眠二
        }
    }
    return score;
}
```

### 3. 胜负判断逻辑
```c
// 检查是否形成五连珠
int check_win(int x, int y)
{
    for (四个方向)
    {
        int count = 1;
        // 正向检查
        for (int i = 1; i < 5; i++)
        {
            if (棋盘边界检查 || 棋子不连续)
            { 
                break;
            }
            count++;
        }
        // 反向检查
        for (int i = 1; i < 5; i++)
        {
            if (棋盘边界检查 || 棋子不连续)
            {
                break;
            } 
            count++;
        }
        // 五连珠
        if (count >= 5)
        {
            return 1;
        }
    }
    return 0;
}
```

## 项目结构
- `五子棋 copy 3.c` - 主程序文件（当前使用版本）
- 其他`五子棋 copy*.c`文件为开发过程中的不同版本

## 注意事项
1. 编译时需要链接数学库（-lm）
2. 确保输入坐标在棋盘范围内
3. AI思考时间随棋盘大小和搜索深度增加
