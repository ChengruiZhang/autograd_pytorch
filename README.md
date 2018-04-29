# Autograd in PyTorch

This is a re-implementation of PyTorch's autograd (`torch.autograd`). 

As you know, Pytorch contains 3 major components:
+ `tensor` can be seen as replacement of `numpy` for both GPU and CPU because it has unified api
+ `autograd.Variable` automatic differentiation tool given a forward formulation
+ `nn`: is deep learning framework build based on `tensor` and `autograd`

This project is aimed to re-implement the `autograd` part because 
+ Pytorch's autograd functions are mostly implemented in `C/C++` so it is much harder for create new autograd's function.
+ Instead of building the backward function of complex function, we can build just the forward function, the backward function will be automatically built based on autograd.
+ Understand clearly how autograd works is very important for serious deep learning learner. 

The reasons why we choose pytorch tensor over numpy array:
+ Pytorch tensor supports GPU
+ We can easily compared your own autograd function with Pytorch's autograd with the same api
+ Getting familiar ourselves with Pytorch's api is a big plus

----

Overview of autograd mechanism:

![](https://cdn-images-1.medium.com/max/1600/1*wE1f2i7L8QRw8iuVx5mOpw.png)

In `variables.py`:
+ `data` is the pytorch tensor
+ `grad` is the Variable contains the gradient of current Variable
+ `grad_fn` is the Function creating the current Variable, if `grad_fn = None`, it is the leaf node of the graph
+ `backward()` receives the `in_grad` from the function that current Variable is used, and call the function that creates that Variable.
+ Other methods for overloading behavior of Variable

In `base\base_function.py` and `base\functions_class.py`:
+ `forward()` receives the `input` and compute the `output`
+ `backward()` reveives the `in_grad` from the Variable it creates and compute the `out_grad` for each its `input` component


In `functions.py`:
+ Interface for building functions
+ Contains the most common of Pytorch tensor's function (not all), but enough for building any graph in Deep Learning

In `eval.py`:
+ `check_example()`: test case for checking your own autograd function is working properly

---

If you want to define your own autograd functions, do the following step:
+ If your autograd function are made up of existing functions (in `functions.py`) or classes in `functions_class.py`, just reuse them.
+ First, define your new function class in `functions_class.py`, the main purpose of each function:
    - `forward`: store `input`, compute `output` and store it to `self.output`, and store other useful information for backward pass
    - `backward`: store `in_grad`, compute `grad` for each `input`, multiply `in_grad` with `grad` to get `out_grad` (according to chain rule) and call each `input` variable's `backward()` function to back-propagate until the leaf node.
+ Second, define your interface in `functions.py` see other functions for more information
+ Third, if the `Variable` supports that function, define new method in `Variable` class in `variables.py`
+ Finally, test the new function with your own test case in `eval.py`