[PythonApplication1.py](https://github.com/user-attachments/files/24163345/PythonApplication1.py)import tkinter as tk
from tkinter import messagebox
from collections import deque
import random

goal = "123456780"
CELL_SIZE = 80  

def moves(state):
    idx = state.index("0")
    result = []
    dirs = {
        0:[1,3], 1:[0,2,4], 2:[1,5],
        3:[0,4,6], 4:[1,3,5,7], 5:[2,4,8],
        6:[3,7], 7:[4,6,8], 8:[5,7]
    }
    for d in dirs[idx]:
        s = list(state)
        s[idx], s[d] = s[d], s[idx]
        result.append("".join(s))
    return result

def bfs(start):
    q = deque([start])
    parent = {start: None}

    while q:
        cur = q.popleft()

        if cur == goal:
        
            path = []
            while cur:
                path.append(cur)
                cur = parent[cur]
            return path[::-1]

        for nxt in moves(cur):
            if nxt not in parent:
                parent[nxt] = cur
                q.append(nxt)

    return None


class PuzzleUI:
    def __init__(self, root):
        self.root = root
        self.state = self.shuffle(goal)
        self.canvas = tk.Canvas(root, width=3*CELL_SIZE, height=3*CELL_SIZE)
        self.canvas.grid(row=0, column=0, columnspan=3)
        self.tiles = {}
        self.draw_board()

        solve_btn = tk.Button(root, text="Solve with BFS", command=self.solve_bfs)
        solve_btn.grid(row=1, column=0, sticky="nsew")
        shuffle_btn = tk.Button(root, text="Shuffle", command=self.shuffle_board)
        shuffle_btn.grid(row=1, column=2, sticky="nsew")

        self.canvas.bind("<Button-1>", self.on_click)

    def shuffle(self, s):
        x = s
        for _ in range(20):
            x = random.choice(moves(x))
        return x

    def draw_board(self):
        self.canvas.delete("all")
        self.tiles = {}
        for i, val in enumerate(self.state):
            if val != "0": 
                row, col = divmod(i, 3)
                x1, y1 = col*CELL_SIZE, row*CELL_SIZE
                x2, y2 = x1+CELL_SIZE, y1+CELL_SIZE
                rect = self.canvas.create_rectangle(x1, y1, x2, y2, fill="skyblue", outline="black")
                text = self.canvas.create_text(x1+CELL_SIZE/2, y1+CELL_SIZE/2, text=val, font=("Arial", 24))
                self.tiles[i] = (rect, text)

    def animate_move_tile(self, index, target_index, callback=None):
     
        if index not in self.tiles:
         
            index = target_index

        row, col = divmod(index, 3)
        t_row, t_col = divmod(target_index, 3)
        dx = (t_col - col) * CELL_SIZE / 10
        dy = (t_row - row) * CELL_SIZE / 10
        steps = 10
        count = 0

        rect, text = self.tiles[index]

        def step():
            nonlocal count
            if count < steps:
                self.canvas.move(rect, dx, dy)
                self.canvas.move(text, dx, dy)
                count += 1
                self.root.after(30, step)
            else:
                lst = list(self.state)
                lst[target_index], lst[index] = lst[index], lst[target_index]
                self.state = "".join(lst)
                self.draw_board()
                if callback:
                    callback()

        step()

    def on_click(self, event):
        col = event.x // CELL_SIZE
        row = event.y // CELL_SIZE
        index = row*3 + col
        zero = self.state.index("0")

        neighbors = {
            0:[1,3], 1:[0,2,4], 2:[1,5],
            3:[0,4,6], 4:[1,3,5,7], 5:[2,4,8],
            6:[3,7], 7:[4,6,8], 8:[5,7]
        }

        if index in neighbors[zero]:
            self.animate_move_tile(index, zero, callback=self.check_goal)

    def check_goal(self):
        if self.state == goal:
            messagebox.showinfo("Congratulations", "You solved the puzzle!")

    def animate_bfs_step(self, prev, curr, callback):
        zero_prev = prev.index("0")
        zero_curr = curr.index("0")
        moved_tile_index = zero_curr
        self.state = prev
        self.animate_move_tile(moved_tile_index, zero_prev, callback=callback)

    def solve_bfs(self):
        path = bfs(self.state)
        if not path:
            messagebox.showerror("Error", "No solution found")
            return

        def animate_path(step=1):
            if step < len(path):
                prev = path[step-1]
                curr = path[step]
                self.animate_bfs_step(prev, curr, callback=lambda s=step+1: animate_path(s))
        animate_path()

    def shuffle_board(self):
        self.state = self.shuffle(goal)
        self.draw_board()


root = tk.Tk()
root.title("8-Puzzle Game GUI with Linked List")
app = PuzzleUI(root)
root.mainloop()


