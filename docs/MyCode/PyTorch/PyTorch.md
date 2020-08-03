# PyTorch

PyTorch实现的Regression 回归：

```python
# Regression 回归
import torch
import matplotlib.pyplot as plt
import torch.nn.functional as F
# from torch.autograd import Variable

x=torch.unsqueeze(torch.linspace(-1,1,100),dim=1) # x data (tensor), shape=(100,1)
y=x.pow(2)+0.2*torch.rand(x.size())  # noisy y data (tensor) ,shape=(100,1)

# plt.scatter(x,y)
#  plt.scatter(x.data.numpy(),y.data.numpy())  似乎并不需要再转换为numpy数据
# plt.show()

class Net(torch.nn.Module):
    def __init__(self, n_feature, n_hidden, n_output):
        super(Net,self).__init__()
        self.hidden=torch.nn.Linear(n_feature, n_hidden)
        self.predict=torch.nn.Linear(n_hidden,n_output)

    def forward(self,x):
        x=torch.relu(self.hidden(x))
        x=self.predict(x)
        return x
net=Net(1,10,1)
print(net)

plt.ion()
plt.show()

optimizer=torch.optim.SGD(net.parameters(),lr=0.5)
loss_func=torch.nn.MSELoss()

for t in range(100):
    prediction=net(x)

    loss=loss_func(prediction,y)

    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
    if t%5==0:
        # plot and show learning process
        plt.cla()
        plt.scatter(x,y)
        plt.plot(x.data.numpy(),prediction.data.numpy(),'r-',lw=5)
        plt.text(0.5,0,'Loss=%.4f'%loss.data,fontdict={'size':20,'color':'red'})
        plt.pause(0.2)

plt.ioff()
plt.show()
```

