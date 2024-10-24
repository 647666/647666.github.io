<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
    <style>
        .python{
            font-weight: 1000;
            font-size:1.5em;
        }
    </style>
</head>
<body>
    <script>
        // 我是单行注释

        /*
            我是多行注释
        */


        /******************   输出语句  ***************/

        // document.write:给网页写入内容,能识别标签.字符需要用引号包裹,纯数字不用

        document.write("实现的是机器人到货物和泊口的路径算法，船舶的分配算法")

        // alert : 弹窗,让网页显示一个弹窗
        alert('各位学长你们好，这里是大一写的第一个ptyhon项目')

        // console.log: 在控制台输出日志(给程序员看,内容不显示在网页,而是在控制台显示,一般是用于调试测试数据)
        console.log('祝大家周末愉快')
        console.log(70123456)
        
        /******************   输入语句  ***************/
        
		//prompt: 弹出输入框,输入内容. 输入框可以放提示文本
        
    </script>
    <pre class="python">
        import sys
        import heapq
        import math
        import logging

        n = 200
        robot_num = 10
        berth_num = 10
        N = 201
        # 设置日志配置：日志文件名，日志格式，日志级别
        logging.basicConfig(filename='robot_path_logs.txt', filemode='w', format='%(asctime)s - %(message)s',
                            level=logging.INFO)


        class Robot:列表来存储每个机器人的路径
        robot_paths_berth = [[] for _ in range(robot_num)]

        obs = 1
        oce = 2
            def __init__(self, startx=0, starty=0, goods=0, status=0, mbx=0, mby=0):
                self.x = startx
                self.y = starty
                self.goods = goods
                self.status = status
                self.mbx = mbx
                self.mby = mby

            def get(self):  # 假设机器人在当前位置获取gds值作为货物
                self.goods = gds[self.x][self.y]


        robot = [Robot() for _ in range(robot_num + 10)]


        class Berth:
            def __init__(self, x=0, y=0, transport_time=0, loading_speed=0):
                self.x = x
                self.y = y
                self.transport_time = transport_time
                self.loading_speed = loading_speed


        berth = [Berth() for _ in range(berth_num + 10)]


        class Boat:
            def __init__(self, num=0, pos=0, status=0):
                self.num = num
                self.pos = pos
                self.status = status


        berth_goods_count = [0] * 10
        used_berths = []
        # 初始化船只与泊位ID的对应关系
        boat_to_berth_id = {}
        # 机器人当前目标的泊位ID
        robot_to_berth_id = {}
        # 已经使用过的泊位ID集合
        used_berths_robot = set()
        robot_positions = {}

        boat = [Boat() for _ in range(10)]
        robot_paths = [[] for _ in range(robot_num)]  # 初始化一个
        money = 0
        boat_capacity = 0
        id = 0
        ch = []
        gds = [[0 for _ in range(N)] for _ in range(N)]
        path = []
        directions = [[] for _ in range(robot_num)]
        targets_berth = []


        def Init():
            for i in range(0, n):
                line = input()
                ch.append([c for c in line])
            for x, row in enumerate(ch):
                for y, char in enumerate(row):
                    if char == '#':
                        gds[x][y] = 1
                    elif char == '*':
                        gds[x][y] = 2
                    elif char == 'B':
                        gds[x][y] = 3
            for i in range(berth_num):
                line = input()
                berth_list = [int(c) for c in line.split(sep=" ")]
                id = berth_list[0]
                berth[id].x = berth_list[1]
                berth[id].y = berth_list[2]
                berth[id].transport_time = berth_list[3]
                berth[id].loading_speed = berth_list[4]
            boat_capacity = int(input())
            okk = input()
            print("OK")
            sys.stdout.flush()


        def Input():
            id, money = map(int, input().split(" "))
            num = int(input())
            for i in range(num):
                x, y, val = map(int, input().split())
                gds[x][y] = val
            for i in range(robot_num):
                robot[i].goods, robot[i].x, robot[i].y, robot[i].status = map(int, input().split())
            for i in range(5):
                boat[i].status, boat[i].pos = map(int, input().split())
                # logging.info(f"boat{i}.status={boat[i].status},boat{i}.pos={boat[i].pos}")
            okk = input()
            return id


        def calculate_distance(robot_pos, target_pos):
            """计算两点之间的距离"""
            return math.sqrt((robot_pos[0] - target_pos[0]) ** 2 + (robot_pos[1] - target_pos[1]) ** 2)


        def find_nearest_target(robot_pos, targets):
            """找到离机器人最近的点"""
            min_distance = float('inf')
            nearest_target = None

            for target in targets:
                target_distance = calculate_distance(robot_pos, target)
                if target_distance < min_distance:
                    min_distance = target_distance
                    nearest_target = target

            return nearest_target


        def find_nearest_target_with_id(robot_pos, targets, used_berths_robot):
            min_distance = float('inf')
            nearest_target = None
            nearest_target_id = None

            for target in targets:
                target_pos = (target[0], target[1])
                target_id = target[2]
                if target_id not in used_berths_robot:
                    distance = calculate_distance(robot_pos, target_pos)

                    if distance < min_distance:
                        min_distance = distance
                        nearest_target = target_pos
                        nearest_target_id = target_id

            return nearest_target, nearest_target_id


        def astar(start, goal, grid, robot_paths, robot_paths_berth, robot_positions, robot_id):
            """
            A*算法实现
            :param start: 起始位置 (x, y)
            :param goal: 目标位置 (x, y)
            :param grid: 游戏地图，0表示可通行，1表示障碍物
            :return: 最短路径
            """

            def heuristic(a, b):
                """
                启发式函数，这里使用欧几里得距离
                """
                return ((a[0] - b[0]) ** 2 + (a[1] - b[1]) ** 2) ** 0.5

            def get_neighbors(current):

                directions = [(0, 1), (1, 0), (0, -1), (-1, 0)]  # 可能的移动方向：上下左右
                neighbors = []
                for dx, dy in directions:
                    nx, ny = current[0] + dx, current[1] + dy
                    if (
                            0 <= nx < len(grid)
                            and 0 <= ny < len(grid[0])
                            and (grid[nx][ny] == 0 or grid[nx][ny] > 2)
                            and (nx, ny) not in robot_paths
                            and (nx, ny) not in robot_paths_berth
                            and all((nx, ny) != pos for rid, pos in robot_positions.items() if rid != robot_id)
                    # 排除除了当前机器人之外的所有机器人的位置
                    ):
                        neighbors.append((nx, ny))
                return neighbors

            frontier = []
            heapq.heappush(frontier, (0 + heuristic(start, goal), start))
            came_from = {start: None}
            cost_so_far = {start: 0}

            while frontier:
                current_priority, current = heapq.heappop(frontier)

                if current == goal:
                    break

                for next in get_neighbors(current):
                    new_cost = cost_so_far[current] + 1  # 假设移动成本是固定的
                    if next not in cost_so_far or new_cost < cost_so_far[next]:
                        cost_so_far[next] = new_cost
                        priority = new_cost + heuristic(goal, next)
                        heapq.heappush(frontier, (priority, next))
                        came_from[next] = current

            # 重建路径
            path = []
            current = goal
            while current in came_from:  # 修改这里的条件，确保不会因为None而卡死
                path.append(current)
                current = came_from[current]  # 使用.get()来避免KeyError
            path.reverse()
            return path


        def path_to_directions(path):
            """
            将路径转换为方向指令
            """
            direction_s = {
                (0, 1): 0,  # 右
                (0, -1): 1,  # 左
                (-1, 0): 2,  # 上
                (1, 0): 3  # 下

            }
            direction_s_list = []
            for i in range(len(path) - 1):
                current, next_ = path[i], path[i + 1]
                dx, dy = next_[0] - current[0], next_[1] - current[1]
                direction_s_list.append(direction_s.get((dx, dy), -1))  # 如果不是合法移动，则使用-1表示
            return direction_s_list


        def select_target_berth():
            global used_berths
            best_berth_id = 11
            shortest_time = float('inf')
            highest_goods_count = -1
            for i in range(berth_num):
                if i not in used_berths:  # 确保不重复选择泊位
                    goods_count = berth_goods_count[i]
                    if goods_count > highest_goods_count or \
                            (goods_count == highest_goods_count and berth[i].transport_time < shortest_time):
                        best_berth_id = i
                        highest_goods_count = goods_count
                        shortest_time = berth[i].transport_time

            if best_berth_id != 11:
                used_berths.append(best_berth_id)  # 更新已使用的泊位列表
                logging.info(f"used_berths={used_berths}")
            return best_berth_id


        def is_in_berth_range(x, y, berth_info):
            for berth in berth_info:
                # 假设每个泊口是4x4大小
                if berth.x <= x < berth.x + 4 and berth.y <= y < berth.y + 4:
                    return True
            return False


        if __name__ == "__main__":
            Init()
            for zhen in range(1, 15001):
                id = Input()
                for i in range(robot_num):
                    # 生成20x20范围内的目标列表
                    robot_positions[i] = (robot[i].x, robot[i].y)
                    targets = [(x, y) for x in range(max(0, robot[i].x - 100), min(len(gds), robot[i].x + 101))
                            for y in range(max(0, robot[i].y - 100), min(len(gds[0]), robot[i].y + 101))
                            if gds[x][y] > 2 and not is_in_berth_range(x, y, berth)]

                    # 如果targets为空，则跳过此次循环
                    if not targets:
                        continue

                    if robot[i].goods == 0:
                        if not directions[i]:
                            # 这里可以根据需要执行特定的逻辑，例如跳过当前循环迭代
                            # 找到最近的目标
                            target = find_nearest_target((robot[i].x, robot[i].y), targets)
                            # 检查目标是否在范围内
                            if (0 <= target[0] < len(gds)) and (0 <= target[1] < len(gds[0])):
                                # 对找到的目标使用A*算法寻找路径
                                path = astar((robot[i].x, robot[i].y), target, gds, robot_paths, robot_paths_berth,
                                            robot_positions, i)
                                robot_paths[i] = path
                                directions[i] = path_to_directions(path)
                                # robot_paths[i].pop(0)
                            if robot[i].x == target[0] and robot[i].y == target[1]:
                                gds[target[0]][target[1]] = 0

                        # 输出移动指令

                        for idx in range(len(directions[i])):
                            next_step = directions[i].pop(0)
                            # robot_paths[i].pop(0)
                            if next_step == 3:  # 下
                                print("move", i, 3)
                                # logging.info(f"move{i} 3")
                                sys.stdout.flush()
                            elif next_step == 2:  # 上
                                print("move", i, 2)
                                # logging.info(f"move{i} 2")
                                sys.stdout.flush()
                            elif next_step == 0:  # 右
                                print("move", i, 0)
                                # logging.info(f"move{i} 0")
                                sys.stdout.flush()
                            elif next_step == 1:  # 左
                                print("move", i, 1)
                                # logging.info(f"move{i} 1")
                                sys.stdout.flush()
                        if not directions[i]:
                            print("get", i)
                            robot_paths[i] = None
                            sys.stdout.flush()

                    if robot[i].goods == 1:
                        if not directions[i]:
                            # 直接使用berth数组中的坐标
                            targets_berth = [(berth[j].x, berth[j].y, j) for j in range(10)]  # 注意，这里加入了泊位ID作为第三个元素
                            targets_berth_coordinate = [(berth[j].x, berth[j].y) for j in range(10)]
                            # 找到最近的目标泊位，同时获取对应的泊位ID
                            nearest_berth, nearest_berth_id = find_nearest_target_with_id((robot[i].x, robot[i].y),
                                                                                        targets_berth, used_berths_robot)

                            # 使用A*算法寻找到最近泊位的路径
                            path = astar((robot[i].x, robot[i].y), nearest_berth, gds, robot_paths, robot_paths_berth,
                                        robot_positions, i)
                            robot_paths_berth[i] = path
                            directions[i] = path_to_directions(path)
                            # robot_paths_berth[i].pop(0)
                            berth_goods_count[nearest_berth_id] += 1
                        # 输出移动指令
                        for idx in range(len(directions[i])):
                            next_step = directions[i].pop(0)
                            if next_step == 3:  # 下
                                print("move", i, 3)
                                # logging.info(f"move{i} 3")
                                sys.stdout.flush()
                            elif next_step == 2:  # 上
                                print("move", i, 2)
                                # logging.info(f"move{i} 2")
                                sys.stdout.flush()
                            elif next_step == 0:  # 右
                                print("move", i, 0)
                                # logging.info(f"move{i} 0")
                                sys.stdout.flush()
                            elif next_step == 1:  # 左
                                print("move", i, 1)
                                # logging.info(f"move{i} 1")
                                sys.stdout.flush()
                        if not directions[i]:
                            print("pull", i)
                            robot_paths_berth[i] = None
                            sys.stdout.flush()

                            # 检查这个机器人是否有对应的泊位ID，并且从已使用泊位中移除
                            if i in robot_to_berth_id:
                                berth_id = robot_to_berth_id[i]
                                if berth_id in used_berths_robot:
                                    used_berths_robot.remove(berth_id)
                                del robot_to_berth_id[i]  # 从记录中删除机器人的泊位ID

                for boat_id in range(5):  # 假设有5艘船
                    if boat[boat_id].status == 1 or boat[boat_id].status == 2:  # 如果船处于泊位外等待状态
                        if boat[boat_id].pos == -1:  # 如果船的位置是 -1，表示它在等待区
                            target_berth_id = select_target_berth()
                            boat_to_berth_id[boat_id] = target_berth_id  # 更新对应关系
                            print("ship", boat_id, target_berth_id)
                            sys.stdout.flush()
                            # logging.info(f"Boat {boat_id} moving to berth {target_berth_id}")
                            boat[boat_id].loading_counter = 0  # 重置装载时间计数器

                        else:  # 船不在等待区，正在向泊位移动
                            target_berth_id = boat_to_berth_id.get(boat_id, None)  # 获取对应的泊位ID
                            if target_berth_id is not None and boat[boat_id].loading_counter >= berth_goods_count[
                                target_berth_id] / berth[target_berth_id].loading_speed:
                                print("go", boat_id)
                                # logging.info(f"remove={target_berth_id}")
                                if target_berth_id in used_berths:
                                    used_berths.remove(target_berth_id)
                                    # logging.info(f"used_berth_remove={used_berths}")
                                berth_goods_count[target_berth_id] = 0
                                del boat_to_berth_id[boat_id]  # 移除船只与泊位ID的对应关系
                            else:
                                # 如果还没有等待足够的帧数，则增加装载时间计数器
                                boat[boat_id].loading_counter += 1

                print("OK")
                sys.stdout.flush()
                highest_goods_count = -1
    </pre>
</body>
</html>
