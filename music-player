import os
import tkinter as tk
from tkinter import filedialog, messagebox
from tkinter import ttk
import pygame
from pygame import mixer
import random
import hashlib

# 初始化pygame混音器
pygame.mixer.init()

# pyinstaller --onefile --icon=music.ico --noconsole main.py
# 支持的音频文件格式
SUPPORTED_FORMATS = ('.mp3', '.wav', '.flac')


class MusicPlayer:
    def __init__(self, root):
        self.root = root
        self.root.title("横川-音乐播放器")
        self.root.geometry("1000x600")

        # 初始化变量
        self.playlist = []
        self.playlist_md5 = set()  # 用于存储播放列表中歌曲的 MD5 值
        self.favorites = []
        self.current_song_index = -1
        self.is_playing = False
        self.total_length = 0
        self.paused = False
        self.play_mode = 0  # 0: 列表顺序播放, 1: 单曲循环, 2: 随机播放
        self.volume = 0.3  # 初始音量为 0.3
        self.current_time = 0  # 新增变量，用于记录当前播放时间

        # 顶部操作区域框架
        top_frame = tk.Frame(self.root)
        top_frame.pack(side=tk.TOP, fill=tk.X, padx=5, pady=5)

        # 新增：歌曲名称搜索框
        search_frame = tk.Frame(top_frame)
        search_frame.pack(pady=5)

        self.search_result_indices = []  # 新增变量，用于记录搜索结果在原播放列表中的索引

        self.search_var = tk.StringVar()
        self.search_var.set("请输入歌曲名称")
        self.search_entry = tk.Entry(search_frame, width=30, textvariable=self.search_var)
        self.search_entry.pack(side=tk.LEFT, padx=5)
        self.search_entry.bind("<FocusIn>", self.on_entry_focus_in)
        self.search_entry.bind("<FocusOut>", self.on_entry_focus_out)
        self.search_entry.bind("<KeyRelease>", self.search_music)

        # 显示当前播放歌曲名称的标签
        self.current_song_label = tk.Label(top_frame, text="当前播放：无")
        self.current_song_label.pack(pady=5)

        # 进度条和时长显示框架
        progress_info_frame = tk.Frame(top_frame)
        progress_info_frame.pack(pady=5)

        # 播放进度条
        self.progress_bar = ttk.Progressbar(progress_info_frame, orient="horizontal", length=800, mode="determinate")
        self.progress_bar.pack(side=tk.LEFT, padx=5)
        self.progress_bar.bind("<Button-1>", self.on_progress_bar_click)

        # 显示已播放时长和总时长的标签
        self.time_label = tk.Label(progress_info_frame, text="0:00 / 0:00")
        self.time_label.pack(side=tk.LEFT)

        # 控件初始化
        button_frame = tk.Frame(top_frame)
        button_frame.pack(pady=5)

        self.add_button = ttk.Button(button_frame, text="导入音乐", width=10, command=self.load_music)
        self.add_button.pack(side=tk.LEFT, padx=5)

        self.previous_button = ttk.Button(button_frame, text="上一首", width=10, command=self.previous_song)
        self.previous_button.pack(side=tk.LEFT, padx=5)

        # 合并播放和暂停按钮
        self.play_pause_button = ttk.Button(button_frame, text="播放", width=10, command=self.play_pause_song)
        self.play_pause_button.pack(side=tk.LEFT, padx=5)

        self.next_button = ttk.Button(button_frame, text="下一首", width=10, command=self.next_song)
        self.next_button.pack(side=tk.LEFT, padx=5)

        self.play_mode_button = ttk.Button(button_frame, text="列表顺序播放", width=15, command=self.change_play_mode)
        self.play_mode_button.pack(side=tk.LEFT, padx=5)

        self.favorite_button = ttk.Button(button_frame, text="收藏", width=10, command=self.add_to_favorites)
        self.favorite_button.pack(side=tk.LEFT, padx=5)

        # 音量调节滑动条
        self.volume_scale = tk.Scale(button_frame, from_=0, to=100, orient=tk.HORIZONTAL, length=150,
                                     command=self.set_volume, showvalue=0)
        self.volume_scale.set(int(self.volume * 100))
        self.volume_scale.pack(side=tk.LEFT, padx=5)
        volume_label = tk.Label(button_frame, text="音量")
        volume_label.pack(side=tk.LEFT)



        # 创建左右两部分框架
        middle_frame = tk.Frame(self.root)
        middle_frame.pack(fill=tk.BOTH, expand=True, padx=5, pady=5)

        self.left_frame = tk.Frame(middle_frame)
        self.left_frame.pack(side=tk.LEFT, fill=tk.BOTH, expand=True, padx=5, pady=5)

        self.right_frame = tk.Frame(middle_frame)
        self.right_frame.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True, padx=5, pady=5)

        # 左侧本地音乐标题
        local_title = tk.Label(self.left_frame, text="本地音乐列表")
        local_title.pack()

        # 左侧本地音乐 Treeview 及滚动条
        local_tree_frame = tk.Frame(self.left_frame)
        local_tree_frame.pack(fill=tk.BOTH, expand=True)

        self.local_playlist_tree = ttk.Treeview(local_tree_frame, columns=('序号', '歌曲名称'), show='headings')
        self.local_playlist_tree.heading('序号', text='序号')
        self.local_playlist_tree.heading('歌曲名称', text='歌曲名称')
        self.local_playlist_tree.column('序号', width=10)
        self.local_playlist_tree.column('歌曲名称', width=340)
        self.local_playlist_tree.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
        self.local_playlist_tree.bind("<Double-1>", self.play_from_local_list)

        local_scrollbar = tk.Scrollbar(local_tree_frame, command=self.local_playlist_tree.yview)
        local_scrollbar.pack(side=tk.RIGHT, fill=tk.Y)
        self.local_playlist_tree.config(yscrollcommand=local_scrollbar.set)

        # 右侧收藏音乐标题
        favorite_title = tk.Label(self.right_frame, text="收藏音乐列表")
        favorite_title.pack()

        # 右侧收藏音乐 Treeview 及滚动条
        favorite_tree_frame = tk.Frame(self.right_frame)
        favorite_tree_frame.pack(fill=tk.BOTH, expand=True)

        self.favorite_playlist_tree = ttk.Treeview(favorite_tree_frame, columns=('序号', '歌曲名称'), show='headings')
        self.favorite_playlist_tree.heading('序号', text='序号')
        self.favorite_playlist_tree.heading('歌曲名称', text='歌曲名称')
        self.favorite_playlist_tree.column('序号', width=10)
        self.favorite_playlist_tree.column('歌曲名称', width=340)
        self.favorite_playlist_tree.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
        self.favorite_playlist_tree.bind("<Double-1>", self.play_from_favorite_list)
        self.favorite_playlist_tree.bind("<Button-3>", self.show_context_menu)  # 绑定右键点击事件

        favorite_scrollbar = tk.Scrollbar(favorite_tree_frame, command=self.favorite_playlist_tree.yview)
        favorite_scrollbar.pack(side=tk.RIGHT, fill=tk.Y)
        self.favorite_playlist_tree.config(yscrollcommand=favorite_scrollbar.set)

        # 创建右键菜单
        self.context_menu = tk.Menu(self.root, tearoff=0)
        self.context_menu.add_command(label="取消收藏", command=self.remove_from_favorites)

        self.update_progress()

    def calculate_md5(self, file_path):
        """计算文件的 MD5 哈希值"""
        hash_md5 = hashlib.md5()
        with open(file_path, "rb") as f:
            for chunk in iter(lambda: f.read(4096), b""):
                hash_md5.update(chunk)
        return hash_md5.hexdigest()

    def load_music(self):
        folder_path = filedialog.askdirectory()
        if folder_path:
            start_index = len(self.playlist) + 1
            new_songs_count = 0
            for file in os.listdir(folder_path):
                if any(file.lower().endswith(ext) for ext in SUPPORTED_FORMATS):
                    file_path = os.path.join(folder_path, file)
                    file_md5 = self.calculate_md5(file_path)
                    if file_md5 not in self.playlist_md5:
                        self.playlist.append(file_path)
                        self.playlist_md5.add(file_md5)
                        self.local_playlist_tree.insert('', 'end', values=(start_index, file))
                        start_index += 1
                        new_songs_count += 1
            messagebox.showinfo("信息", f"已导入 {new_songs_count} 首新音乐")

    def play_pause_song(self):
        if not self.playlist and not self.favorites:
            messagebox.showwarning("警告", "没有音乐可播放！")
            return

        if self.paused:
            mixer.music.unpause()
            self.paused = False
            self.play_pause_button.config(text="暂停")
            self.is_playing = True
        elif not self.is_playing:
            if self.current_song_index == -1:
                if self.playlist:
                    self.current_song_index = 0
                elif self.favorites:
                    self.current_song_index = 0
                    self.playlist = self.favorites
            song = self.playlist[self.current_song_index]
            try:
                mixer.music.load(song)
                mixer.music.set_volume(self.volume)
                mixer.music.play(start=self.current_time)
                self.is_playing = True
                self.play_pause_button.config(text="暂停")
                self.total_length = mixer.Sound(song).get_length()
                song_name = os.path.basename(song)
                self.current_song_label.config(text=f"当前播放：{song_name}")
            except pygame.error as e:
                messagebox.showerror("错误", f"无法播放 {song_name}：{e}")
        else:
            mixer.music.pause()
            self.paused = True
            self.play_pause_button.config(text="播放")
            self.is_playing = False

    def previous_song(self):
        if self.playlist or self.favorites:
            if self.play_mode == 0:  # 列表顺序播放
                if self.current_song_index > 0:
                    self.current_song_index -= 1
                else:
                    if self.playlist:
                        self.current_song_index = len(self.playlist) - 1
                    elif self.favorites:
                        self.current_song_index = len(self.favorites) - 1
            elif self.play_mode == 1:  # 单曲循环
                if self.playlist:
                    self.current_song_index = (self.current_song_index - 1) % len(self.playlist)
                elif self.favorites:
                    self.current_song_index = (self.current_song_index - 1) % len(self.favorites)
            elif self.play_mode == 2:  # 随机播放
                available_indices = list(range(len(self.playlist)))
                if available_indices:
                    available_indices.remove(self.current_song_index)
                    if available_indices:
                        self.current_song_index = random.choice(available_indices)
            self.current_time = 0  # 切换歌曲时重置当前播放时间
            self.play_song()

    def next_song(self):
        if self.playlist or self.favorites:
            if self.play_mode == 0:  # 列表顺序播放
                if self.playlist:
                    self.current_song_index = (self.current_song_index + 1) % len(self.playlist)
                elif self.favorites:
                    self.current_song_index = (self.current_song_index + 1) % len(self.favorites)
            elif self.play_mode == 1:  # 单曲循环
                if self.playlist:
                    self.current_song_index = (self.current_song_index + 1) % len(self.playlist)
                elif self.favorites:
                    self.current_song_index = (self.current_song_index + 1) % len(self.favorites)
            elif self.play_mode == 2:  # 随机播放
                available_indices = list(range(len(self.playlist)))
                if available_indices:
                    available_indices.remove(self.current_song_index)
                    if available_indices:
                        self.current_song_index = random.choice(available_indices)
            self.current_time = 0  # 切换歌曲时重置当前播放时间
            self.play_song()

    # 自动播放结束调用，区别与手动点击下一首。
    def next_song_auto_end(self):
        if self.playlist or self.favorites:
            if self.play_mode == 0:  # 列表顺序播放
                if self.playlist:
                    self.current_song_index = (self.current_song_index + 1) % len(self.playlist)
                elif self.favorites:
                    self.current_song_index = (self.current_song_index + 1) % len(self.favorites)
            elif self.play_mode == 2:  # 随机播放
                available_indices = list(range(len(self.playlist)))
                if available_indices:
                    if len(available_indices) > 1:
                        available_indices.remove(self.current_song_index)
                    self.current_song_index = random.choice(available_indices)
            # 单曲循环模式不改变 self.current_song_index，直接播放当前歌曲

            self.current_time = 0  # 切换歌曲时重置当前播放时间
            self.play_song()

    def add_to_favorites(self):
        selected_item = self.local_playlist_tree.selection()
        if selected_item:
            index = int(self.local_playlist_tree.item(selected_item)['values'][0]) - 1
            song = self.playlist[index]
            if song not in self.favorites:
                self.favorites.append(song)
                song_name = os.path.basename(song)
                self.favorite_playlist_tree.insert('', 'end', values=(len(self.favorites), song_name))
                messagebox.showinfo("信息", "已添加到收藏夹")
            else:
                messagebox.showinfo("信息", "此音乐已在收藏夹中")
        else:
            messagebox.showwarning("警告", "请先选择一首本地音乐！")

    def update_progress(self):
        if self.is_playing:
            if not self.paused:
                self.current_time += 1
            if self.total_length > 0:
                progress = (self.current_time / self.total_length) * 100
                self.progress_bar["value"] = progress

                current_minutes = int(self.current_time // 60)
                current_seconds = int(self.current_time % 60)
                total_minutes = int(self.total_length // 60)
                total_seconds = int(self.total_length % 60)
                current_time_str = f"{current_minutes}:{current_seconds:02d}"
                total_time_str = f"{total_minutes}:{total_seconds:02d}"
                self.time_label.config(text=f"{current_time_str} / {total_time_str}")

            if self.current_time >= self.total_length:  # 歌曲播放结束
                self.current_time = 0
                self.next_song_auto_end()

        self.root.after(1000, self.update_progress)

    def play_from_local_list(self, event):
        selected_item = self.local_playlist_tree.selection()
        if selected_item:
            tree_index = int(self.local_playlist_tree.item(selected_item)['values'][0]) - 1
            if self.search_result_indices:
                self.current_song_index = self.search_result_indices[tree_index]
            else:
                self.current_song_index = tree_index
            self.current_time = 0  # 切换歌曲时重置当前播放时间
            self.playlist = self.playlist
            self.play_song()

    def play_from_favorite_list(self, event):
        selected_item = self.favorite_playlist_tree.selection()
        if selected_item:
            self.current_song_index = int(self.favorite_playlist_tree.item(selected_item)['values'][0]) - 1
            self.current_time = 0  # 切换歌曲时重置当前播放时间
            self.playlist = self.favorites
            self.play_song()

    def play_song(self):
        if self.is_playing:
            mixer.music.stop()
        song = self.playlist[self.current_song_index]
        song_name = os.path.basename(song)
        try:
            mixer.music.load(song)
            mixer.music.set_volume(self.volume)
            mixer.music.play(start=self.current_time)
            self.is_playing = True
            self.paused = False
            self.play_pause_button.config(text="暂停")
            self.total_length = mixer.Sound(song).get_length()
            self.current_song_label.config(text=f"当前播放：{song_name}")
        except pygame.error as e:
            messagebox.showerror("错误", f"无法播放 {song_name}：{e}")

    def on_progress_bar_click(self, event):
        if self.is_playing and self.total_length > 0:
            progress_bar_width = self.progress_bar.winfo_width()
            click_percentage = event.x / progress_bar_width
            self.current_time = click_percentage * self.total_length
            try:
                mixer.music.stop()
                mixer.music.load(self.playlist[self.current_song_index])
                mixer.music.set_volume(self.volume)
                mixer.music.play(start=self.current_time)
            except pygame.error:
                pass
            progress = click_percentage * 100
            self.progress_bar["value"] = progress

            current_minutes = int(self.current_time // 60)
            current_seconds = int(self.current_time % 60)
            total_minutes = int(self.total_length // 60)
            total_seconds = int(self.total_length % 60)
            current_time_str = f"{current_minutes}:{current_seconds:02d}"
            total_time_str = f"{total_minutes}:{total_seconds:02d}"
            self.time_label.config(text=f"{current_time_str} / {total_time_str}")

    def change_play_mode(self):
        self.play_mode = (self.play_mode + 1) % 3
        if self.play_mode == 0:
            self.play_mode_button.config(text="列表顺序播放")
        elif self.play_mode == 1:
            self.play_mode_button.config(text="单曲循环")
        elif self.play_mode == 2:
            self.play_mode_button.config(text="随机播放")

    def set_volume(self, volume_value):
        """设置音量"""
        self.volume = float(volume_value) / 100
        mixer.music.set_volume(self.volume)

    def show_context_menu(self, event):
        """显示右键菜单"""
        selected_item = self.favorite_playlist_tree.identify_row(event.y)
        if selected_item:
            self.favorite_playlist_tree.selection_set(selected_item)
            self.context_menu.post(event.x_root, event.y_root)

    def remove_from_favorites(self):
        """从收藏列表中移除歌曲"""
        selected_item = self.favorite_playlist_tree.selection()
        if selected_item:
            index = int(self.favorite_playlist_tree.item(selected_item)['values'][0]) - 1
            song = self.favorites[index]

            # 若正在播放的歌曲被取消收藏
            if self.playlist == self.favorites and self.current_song_index == index:
                mixer.music.stop()
                self.is_playing = False
                self.paused = False
                self.play_pause_button.config(text="播放")
                self.current_song_label.config(text="当前播放：无")
                self.current_time = 0
                self.progress_bar["value"] = 0
                self.time_label.config(text="0:00 / 0:00")

            # 从收藏列表中移除歌曲
            self.favorites.pop(index)

            # 从 Treeview 中删除该项
            self.favorite_playlist_tree.delete(selected_item)

            # 更新收藏列表中歌曲的序号
            for i, item in enumerate(self.favorite_playlist_tree.get_children()):
                self.favorite_playlist_tree.item(item, values=(i + 1, os.path.basename(self.favorites[i])))

            # 若当前播放列表是收藏列表，更新播放列表索引
            if self.playlist == self.favorites:
                if index < self.current_song_index:
                    self.current_song_index -= 1
                elif index == self.current_song_index and self.favorites:
                    self.current_song_index = 0

            messagebox.showinfo("信息", "已从收藏夹中移除")

    def on_entry_focus_in(self, event):
        if self.search_var.get() == "请输入歌曲名称":
            self.search_var.set("")

    def on_entry_focus_out(self, event):
        if self.search_var.get() == "":
            self.search_var.set("请输入歌曲名称")

    def search_music(self, event):
        keyword = self.search_var.get().lower()
        if keyword == "请输入歌曲名称":
            keyword = ""
        # 清空本地音乐列表的 Treeview
        for item in self.local_playlist_tree.get_children():
            self.local_playlist_tree.delete(item)
        self.search_result_indices = []  # 清空搜索结果索引列表

        if keyword:
            start_index = 1
            for i, song in enumerate(self.playlist):
                song_name = os.path.basename(song).lower()
                if keyword in song_name:
                    self.local_playlist_tree.insert('', 'end', values=(start_index, os.path.basename(song)))
                    self.search_result_indices.append(i)  # 记录索引
                    start_index += 1
        else:
            # 搜索条件清空，展示全部歌曲
            start_index = 1
            for i, song in enumerate(self.playlist):
                self.local_playlist_tree.insert('', 'end', values=(start_index, os.path.basename(song)))
                self.search_result_indices.append(i)  # 记录索引
                start_index += 1

if __name__ == "__main__":
    root = tk.Tk()
    player = MusicPlayer(root)
    root.mainloop()
