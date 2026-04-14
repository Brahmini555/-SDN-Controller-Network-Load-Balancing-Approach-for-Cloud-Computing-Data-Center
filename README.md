# -SDN-Controller-Network-Load-Balancing-Approach-for-Cloud-Computing-Data-Center
mport tkinter

from tkinter import *

import math

import random

from threading import Thread

from collections import defaultdict

from tkinter import ttk

import matplotlib.pyplot as plt

import numpy as np

import time

import random

import networkx as nx

 

global mobile, labels, mobile_x, mobile_y, text, canvas, source_list, root, num_nodes, tf1, nodes, dest_list, shortest_path, propose, existing

option = 0

propose = []

existing = []

 

def getDistance(iot_x,iot_y,x1,y1):

   flag = False

   for i in range(len(iot_x)):

       dist = math.sqrt((iot_x[i] - x1)**2 + (iot_y[i] - y1)**2)

       if dist < 60:

           flag = True

           break

   return flag

 

def generateNetwork():

   global mobile, labels, mobile_x, mobile_y, num_nodes, tf1, nodes

   mobile = []

   mobile_x = []

   mobile_y = []

   labels = []

   nodes = []

   canvas.update()

   num_nodes = int(tf1.get().strip())

   for i in range(0,num_nodes):
 run = True

       while run == True:

           x = random.randint(50, 450)

           y = random.randint(50, 600)

           flag = getDistance(mobile_x,mobile_y,x,y)

           if flag == False:

               nodes.append([x, y])

               mobile_x.append(x)

               mobile_y.append(y)

               run = False

               name = canvas.create_oval(x,y,x+40,y+40, fill="red")

               lbl = canvas.create_text(x+20,y-10,fill="darkblue",font="Times 8 italic bold",text="MN "+str(i))

               labels.append(lbl)

               mobile.append(name)    

 

def calculatePath():

   global source_list, dest_list, nodes, shortest_path, propose, existing

   text.delete('1.0', END)

   src = int(source_list.get())

   dest = int(dest_list.get())

   G = nx.DiGraph()

   for i in range(len(nodes)):

       G.add_node(str(i))        

   for i in range(len(nodes)):

       for j in range(len(nodes)):

           if i != j:

               list1 = nodes[i]

               list2 = nodes[j]

               dist = math.sqrt((list1[0] - list2[0])**2 + (list1[1] - list2[1])**2)

               if dist < 150:

                   G.add_edge(str(i), str(j), weight = dist)                    

   try:

       shortest_path = list(nx.all_shortest_paths(G, str(src), str(dest)))

       choosen_path = None

       less_load = 10000

       max_load = 0

       for i in range(len(shortest_path)):

           load = random.randint(5, len(nodes))

           path = shortest_path[i]

           load = load / len(nodes)

           if load < less_load:

               max_load = less_load

               less_load = load

               choosen_path = path

           text.insert(END,"Path"+str(i+1)+": "+str(path)+" Elephant Load: "+str(load)+"\n")

       if max_load != 10000:

           propose.append([(len(propose) + 1), less_load])

           existing.append([(len(existing) + 1), max_load])

       shortest_path = choosen_path

       text.insert(END,"\nChoosen path with less load = "+str(shortest_path)+"\n\n")

   except Exception:

       text.insert(END,"No Shortest Path found between Source IOT"+str(src)+" & Destination IOT"+str(dest)+"\n")

       text.insert(END,"Please choose some other source and destination")

 

def startDataTransferSimulation(canvas, lines, shortest_path, mobile_x, mobile_y, text, src):

   class SimulationThread(Thread):

       def __init__(self, canvas, lines, shortest_path, mobile_x, mobile_y, text, src):

           Thread.__init__(self)

           self.canvas = canvas

           self.lines = lines

           self.shortest_path = shortest_path

           self.mobile_x = mobile_x

           self.mobile_y = mobile_y

           self.text = text

           self.src = src

       def run(self):

           time.sleep(1)

           for i in range(0,3):

               for k in range(len(self.lines)):

                   self.canvas.delete(self.lines[k])

               time.sleep(1)

               self.lines.clear()

               start = 0

               while start < len(self.shortest_path)-1:

                   end = start + 1

                   src_x = self.mobile_x[int(self.shortest_path[start])]

                   src_y = self.mobile_y[int(self.shortest_path[start])]

                   dest_x = self.mobile_x[int(self.shortest_path[end])]

                   dest_y = self.mobile_y[int(self.shortest_path[end])]

                   line1 = self.canvas.create_line(src_x+20, src_y+20, dest_x+20, dest_y+20, fill='black', width=3)

                   self.lines.append(line1)

                   start += 1

               time.sleep(1)

           for k in range(len(self.lines)):

               self.canvas.delete(self.lines[k])    

           self.canvas.update()            

   newthread = SimulationThread(canvas, lines, shortest_path, mobile_x, mobile_y, text, src)

   newthread.start()

 

def communication():

   global shortest_path

   lines = []

   start = 0

   src = source_list.get()

   while start < len(shortest_path)-1:

       end = start + 1

       src_x = mobile_x[int(shortest_path[start])]

       src_y = mobile_y[int(shortest_path[start])]

       dest_x = mobile_x[int(shortest_path[end])]

       dest_y = mobile_y[int(shortest_path[end])]

       line1 = canvas.create_line(src_x+20, src_y+20, dest_x+20, dest_y+20, fill='black', width=3)

       lines.append(line1)

       start += 1

   canvas.update()

   startDataTransferSimulation(canvas, lines, shortest_path, mobile_x, mobile_y, text, src)

   option = 1

 

def graph():

   p = np.asarray(propose)

   e = np.asarray(existing)

   plt.plot(p[:,0], p[:,1])

   plt.plot(e[:,0], e[:,1])

   plt.legend(['SDN Delay', 'Existing Tail Drop Delay'])

   plt.xlabel("Algorithm Names")

   plt.ylabel("Number of Transmissions")

   plt.title("Delay Comparison Graph")

   plt.show()

 

def Main():

   global root, tf1, text, canvas, source_list, dest_list

   root = tkinter.Tk()

   root.geometry("1300x1200")

   root.title("SDN Controller Network Load Balancing Approach for Cloud Computing Data Center")

   root.resizable(True,True)

   font1 = ('times', 12, 'bold')

 

   canvas = Canvas(root, width = 800, height = 700)

   canvas.pack()

 

   l2 = Label(root, text='Num Nodes:')

   l2.config(font=font1)

   l2.place(x=820,y=10)

 

   tf1 = Entry(root,width=10)

   tf1.config(font=font1)

   tf1.place(x=970,y=10)

 

   generateButton = Button(root, text="Generate SDN Network", command=generateNetwork)

   generateButton.place(x=820,y=60)

   generateButton.config(font=font1)

 

   l1 = Label(root, text='Source Node:')

   l1.config(font=font1)

   l1.place(x=820,y=110)

 

   source = []

   for i in range(0, 50):

       source.append(str(i))

   source_list = ttk.Combobox(root, values=source, postcommand=lambda: source_list.configure(values=source))

   source_list.place(x=970,y=110)

   source_list.current(0)

   source_list.config(font=font1)

 

   l3 = Label(root, text='Destination Node:')

   l3.config(font=font1)

   l3.place(x=820,y=160)

 

   dest = []

   for i in range(0, 50):

       dest.append(str(i))

   dest_list = ttk.Combobox(root, values=dest, postcommand=lambda: dest_list.configure(values=dest))

   dest_list.place(x=970,y=160)

   dest_list.current(0)

   dest_list.config(font=font1)

 

   pathButton = Button(root, text="Calculate Load on Each Path", command=calculatePath)

   pathButton.place(x=820,y=210)

   pathButton.config(font=font1)

 

   dataButton = Button(root, text="Elephant Flow using Light Low Path", command=communication)

   dataButton.place(x=820,y=260)

   dataButton.config(font=font1)

 

   graphButton = Button(root, text="Delay Graph", command=graph)

   graphButton.place(x=820,y=310)

   graphButton.config(font=font1)

 

   text=Text(root,height=18,width=360)

   scroll=Scrollbar(text)

   text.configure(yscrollcommand=scroll.set)

   text.place(x=820,y=360)
   root.mainloop()
if __name__== '__main__' :

   Main ()

   

