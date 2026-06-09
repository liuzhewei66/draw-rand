# draw-rand
simple random lottery draw tool, support custom participants and prize settings
import tkinter as tk
from tkinter import filedialog, messagebox, simpledialog  # 全部放顶部
import random

class LotteryTool:
    def __init__(self, root):
        self.root = root
        self.root.title("随机抽奖工具")
        self.root.geometry("520x420")

        # 数据存储
        self.name_list = []
        self.is_running = False
        self.current_name = ""

        # 控件布局
        frame_top = tk.Frame(root)
        frame_top.pack(pady=8)

        tk.Button(frame_top, text="导入名单TXT", command=self.load_file, width=12).grid(row=0, column=0, padx=5)
        tk.Button(frame_top, text="手动添加名单", command=self.add_manually, width=12).grid(row=0, column=1, padx=5)
        tk.Button(frame_top, text="清空名单", command=self.clear_all, width=12).grid(row=0, column=2, padx=5)

        # 中奖名字大屏
        self.show_label = tk.Label(root, text="等待开始抽奖", font=("黑体", 36), fg="#e63946")
        self.show_label.pack(pady=30)

        # 按钮区
        frame_btn = tk.Frame(root)
        frame_btn.pack()
        self.start_btn = tk.Button(frame_btn, text="开始摇号", command=self.start_roll, font=("微软雅黑",14), width=10, bg="#457b9d", fg="white")
        self.start_btn.grid(row=0, column=0, padx=10)
        self.stop_btn = tk.Button(frame_btn, text="停止开奖", command=self.stop_roll, font=("微软雅黑",14), width=10, bg="#e63946", fg="white", state=tk.DISABLED)
        self.stop_btn.grid(row=0, column=1, padx=10)

        # 名单数量提示
        self.info_label = tk.Label(root, text=f"当前名单总数：{len(self.name_list)} 人")
        self.info_label.pack(pady=15)

    # 导入txt文件，一行一个姓名
    def load_file(self):
        path = filedialog.askopenfilename(filetypes=[("文本文件", "*.txt")])
        if not path:
            return
        try:
            with open(path, "r", encoding="utf-8") as f:
                lines = [n.strip() for n in f.readlines() if n.strip()]
            self.name_list.extend(lines)
            self.update_count()
            messagebox.showinfo("成功", f"导入{len(lines)}条名单")
        except Exception as e:
            messagebox.showerror("错误", f"读取失败：{str(e)}")

    # 手动弹窗添加姓名
    def add_manually(self):
        # 直接用 simpledialog，不用 tk.simpledialog
        name = simpledialog.askstring("添加人员", "输入姓名：")
        if name and name.strip():
            self.name_list.append(name.strip())
            self.update_count()

    # 更新人数显示
    def update_count(self):
        self.info_label.config(text=f"当前名单总数：{len(self.name_list)} 人")

    # 清空所有名单
    def clear_all(self):
        self.name_list.clear()
        self.show_label.config(text="等待开始抽奖")
        self.update_count()

    # 滚动名字
    def roll_name(self):
        if not self.is_running or len(self.name_list) == 0:
            return
        self.current_name = random.choice(self.name_list)
        self.show_label.config(text=self.current_name)
        self.root.after(60, self.roll_name)

    # 开始抽奖
    def start_roll(self):
        if len(self.name_list) == 0:
            messagebox.showwarning("提醒", "请先导入或添加抽奖名单！")
            return
        self.is_running = True
        self.start_btn.config(state=tk.DISABLED)
        self.stop_btn.config(state=tk.NORMAL)
        self.roll_name()

    # 停止开奖
    def stop_roll(self):
        self.is_running = False
        self.start_btn.config(state=tk.NORMAL)
        self.stop_btn.config(state=tk.DISABLED)
        messagebox.showinfo("中奖结果", f"恭喜：{self.current_name}！")

if __name__ == "__main__":
    win = tk.Tk()
    app = LotteryTool(win)
    win.mainloop()
