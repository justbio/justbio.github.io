---
layout: post
category: "read"
title:  "tkinter笔记"
---

import tkinter as tk  

创建窗口  
window = tk.Tk()  
window.title("name")  
window.geometry('heightxwidth')  
top=tk.Toplevel(window)  
window.mainloop() 

关闭窗口  
window.destroy()

<!-- more -->

变量设置  
var = tk.StringVar()  
var = tk.IntVar() 
var.set("var")  

标签  
l=tk.Label(window,textvariable=var,bg="color",font=("Arial",12),width=15,height=2)  
l.pack()  
l.config()

按钮  
b=tk.Button(window,text="text",width=15,height=2,command=comm)  
b.pack()

输入框  
e=tk.Entry(window,show='*')  
e.pack()  
e.get()

文本框  
t=tk.Entry(window,)  
t.pack()  
t.insert('insert/end/row.col',var)

列表  
lb=tk.Listbox(window,listvariable=var)   
lb.insert('insert/end'/row,var)    
lb.delete(row)   
lb.curseselection()

单选框   
r=tk.Radiobutton(window,text="text",variable=var,value="V",command=comm)  

多选框  
c=tk.Checkbutton(window,text="text",variable=var,onvalue=1,offvalue=0,command=comm)
c.pack()

标尺  
s=tk.Scale(window,label="text",from_=n,to=m,orient=tk.HORIZONTAL,lenth=nnn,showvalue=0,tickinterval=x,resolution=0.01,command=comm)  
s.pack()

画布  
canvas=tk.Cancas(window,bg='color',height=n,width=m)  
img_file=tk.PhotoImage(file="file")  
img=canvas.create_image(0,0,anchor="position",image=img_file)  
line=canvas.create_line(x0,y0,x1,y1)  
oval=canvas.create_oval(x0,y0,x1,y1,fill='color')  
arc=canvas.create_arc(x0,y0,x1,y1,start=n,extent=m)  
rect=canvas.create_rectangle(x0,y0,x1,y1)  
canvas.pack()  
canvas.move(rect,m,n)

菜单  
menubar=tk.Menu(window)  
filemenu=tk.Menu(menubar,tearoff=0)   
menubar.add_cascade(label='File',menu=filemenu)   
filemenu.add_command(label='New',command=comm)  
filemenu.add_separator()  
submenu=tk.Menu(filemenu)
window.config(menu=menubar)

框架  
frm=tk.Frame(window)  
frm.pack  
frm_sub=tk.Frame(frm)  
frm_sub.pack('left/right')

弹出窗口  
tk.messagebox.showinfo(title="text",message="text")  
tk.messagebox.showwaring(title="text",message="text")  
tk.messagebox.showerror(title="text",message="text")  
tk.messagebox.askquestion(title="text",message="text")  
tk.messagebox.askyesno(title="text",message="text")  
tk.messagebox.asktrycancel(title="text",message="text")  
tk.messagebox.askokcancel(title="text",message="text")  

放置部件  
.pack(side="up/down/left/right")  
.grid(row=n,column=m,padx=x,pady=y,ipadx=x1,ipady=y1)  
.place(x=n,y=m,anchor="position")

选择文件的窗口  
import tkinter.filedialog  

tkinter.filedialog.askopenfilename()






