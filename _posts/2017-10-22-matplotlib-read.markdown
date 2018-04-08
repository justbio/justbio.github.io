---
layout: post
category: "read"
title:  "matplotlib笔记"
---

创建画布
plt.figure(num=n，figsize=(width,hight))

线图  
plt.plot(x,y,color='color',linewidth=n,linestyle='--',label='name')  

散点图
plt.scatter(x,y,s=n,c='color',alpha=n)

柱状图  
plt.bar(x,y,facecolor'color',edgecolor='color',)

等高线图  
X,Y = np.meshgrid(x,y)  
plt.contourf(X,Y,Z,n,alpha=0.7,cmap=plt.cm.hot)  
C=plt.contour(X,Y,Z,n,color=0.7,linewidth=.5)  
plt.clabel(C,inline=True,fontsize=n)  

图像  
plt.imshow(a,interpolation='nearest',cmap='bone',origin='lower/upper')

坐标轴文字  
plt.xlim((min,max))/plt.ylim((min,max))  
plt.xlabel("discription")/plt.ylabel("discription")
plt.xticks([new_ticks],[new_discription])/
plt.yticks([new_ticks],[new_discription])  

坐标轴样式和移动  
ax=plt.gca()  
ax.spines["right"].set_color("none")  
ax.spines["top"].set_color("none")  
ax.xaixs.set_ticks_position("bottom")  
ax.yaixs.set_ticks_position("left")  
ax.spines["bottom"].set_position(('data',0))
ax.spines["left"].set_position(('data',0))  

for label in ax.get_xticklabels()+ax.get_yticklabels():  
    label.set_fontsize(12)  
    label.set.bbox(dict(facecolor='white',edgecolor='None',alpha=0.7))

图例  
plt.legend(handles=[],labels=[],loc='best')

注解  
plt.annotate('text',xy=(x,y),xycoords='data',xytext=(+n,-n),textcoords="offset points",fontsize=n,arrowprops=dict(arrowstyle='->',connectionstyle='arc3,rad=.2'))  

plt.text(x,y,'text',fontdict={'size':16,'color':'r'},ha='center',va='bottom')

色柱  
plt.colorbar(shrink=n)

显示图  
plt.show()

3D图像  
from mpl_toolkits.mplot3d import Axes3D  
fig =plt.figure()  
ax = Axes3D(fig)  
ax.plot_surface(X,Y,Z,rstride=1,cstride=1,cmap=plt.get_cmap('rainbow'))  
ax.contourf(X,Y,Z,zdir='x/y/z',offset=n,cmap='rainbow')  

Subplot  
plt.subplot(rown,column,num)  

分割显示
import matplotlib.girdspec as gridspec  

Methed 1:  
ax1 = plt.subplot2grid((row,col),(rown,coln)colspan=n,rowspan=n)  
ax1.plot(x,y)  
ax2.......  
Methed 2:  
gs=gridspec.GridSpec(row,col)  
ax1=plt.subplot(gs[row,col])  
ax1.plot(x,y)  
ax2.......  
Mwthed 3:  
f,((ax11,ax12),(ax21,ax22)) = plt.subplots(row,col,sharex=True,sharey=True)  
ax11.plot(x,y)  
ax12.......  

图中图  
Methed 1: 
ax2 = fig.add_axes([left,bottom,width,height])  
Methed 2:  
plt.axes([left,bottom,width,height])

次坐标轴  
fig,ax1=plt.subplots()  
ax2 = ax1.twinx()  

动画  
from matplotlib import animation  
fig,ax = plt.subplots()  
x= np.arange(0,2*np.pi,0.01)  
line, =ax.plot(x,np.sin(x))  

def animate(i):  
    line.set_ydata(np.sin(x+i/10))  
    return line,  
def init():  
    line.set_ydata(np.sin(x))  
    return line,   

ani = animation.FuncAnimation(fig=fig,func=animate,frames=n,init_func=init,interval=n,blit=False)  
