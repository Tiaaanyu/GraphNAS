# GraphNAS

#### Overview
Graph Neural Architecture Search (GraphNAS for short) enables automatic design of the best graph neural architecture 
based on reinforcement learning. 
This directory contains code necessary to run GraphNAS. 
Specifically, GraphNAS first uses a recurrent network to generate variable-length strings 
that describe the architectures of graph neural networks, and then trains the recurrent network 
with a policy gradient algorithm to maximize the expected accuracy of the generated architectures on a validation data set. 
An illustration of GraphNAS is shown as follows:

<p align="center">
<img src="./images/macro_search.png" width="500"  alt="A simple illustration of GraphNAS" align=center>
</p>

A recurrent network (Controller RNN) generates descriptions of graph neural architectures (Child model GNNs). 
Once an architecture ![m](http://latex.codecogs.com/gif.latex?m) is generated by the controller, 
GraphNAS trains the architecture m on a given graph ![G](http://latex.codecogs.com/gif.latex?G) and 
test ![m](http://latex.codecogs.com/gif.latex?m) 
on a validate set ![D](http://latex.codecogs.com/gif.latex?D). 
The validation result ![R_D(m)](http://latex.codecogs.com/gif.latex?R_D(m)) is taken as the reward of the recurrent network.


  
  
  <br/>
  <br/>
To improve the search efficiency of GraphNAS, we restrict the search space from an entire architecture to a 
concatenation of the best search results built on each single architecture layer. 
An example of GraphNAS constructing a single GNN layer (the right-hand side) is shown as follows:

<p align="center">
<img src="./images/micro_search.png" width="500"  alt="A simple illustration of GraphNAS" align=center>
</p> 

In the above example, the layer has two input states ![O_1](http://latex.codecogs.com/gif.latex?O_1) and ![O_2](http://latex.codecogs.com/gif.latex?O_2), two intermediate states ![O_3](http://latex.codecogs.com/gif.latex?O_3) and ![O_4](http://latex.codecogs.com/gif.latex?O_4), 
and an output state ![O_5](http://latex.codecogs.com/gif.latex?O_5). The controller at the left-hand side samples ![O_2](http://latex.codecogs.com/gif.latex?O_2) from ![{O_1, O_2, O_3}](http://latex.codecogs.com/gif.latex?%5C%7BO_1%2C%20O_2%2C%20O_3%5C%7D) 
and takes ![O_2](http://latex.codecogs.com/gif.latex?O_2) as input of ![O_4](http://latex.codecogs.com/gif.latex?O_4), and then samples "GAT" for processing ![O_2](http://latex.codecogs.com/gif.latex?O_2). The output state ![O_5=relu(O_3+O_4)](http://latex.codecogs.com/gif.latex?O_5=relu(O_3+O_4)) 
collects information from ![O_3](http://latex.codecogs.com/gif.latex?O_3) and ![O_4](http://latex.codecogs.com/gif.latex?O_4), and the controller assigns a readout operator "add" and an activation operator 
"relu" for ![O_5](http://latex.codecogs.com/gif.latex?O_5). As a result, this layer can be described as a list of operators: [0, gcn, 1, gat, add, relu].


#### Requirements
Recent versions of PyTorch, numpy, scipy, sklearn, dgl, torch_geometric and networkx are required.
Ensure that at least PyTorch 1.0.0 is installed. Then run:
    
    pip install -r requirements.txt

If you want to run in docker, you can run:

    docker build -t graphnas -f DockerFile . 
    docker run -it -v $(pwd):/GraphNAS graphnas python -m eval_scripts.semi.eval_designed_gnn

#### Running the code
##### Architecture evaluation
To evaluate our best architecture designed on semi-supervised experiments by training from scratch, run

    python -m eval_scripts.semi.eval_designed_gnn

To evaluate our best architecture designed on semi-supervised experiments by training from scratch, run

    python -m eval_scripts.sup.eval_designed_gnn
###### Results
Semi-supervised node classification w.r.t. accuracy

Model| Cora | Citeseer | Pubmed
|-|-|-|-| 
GCN    | 81.5+/-0.4 | 70.9+/-0.5   | 79.0+/-0.4  
SGC    |  81.0+/-0.0 |   71.9+/-0.1   |  78.9+/-0.0   
GAT    |  83.0+/-0.7  |  72.5+/-0.7   | 79.0+/-0.3    
LGCN    |  83.3+/-0.5  | 73.0+/-0.6    |  79.5+/-0.2   
DGCN    |  82.0+/-0.2  | 72.2+/-0.3    |  78.6+/-0.1   
ARMA    |  82.8+/-0.6  | 72.3+/-1.1    |  78.8+/-0.3   
APPNP   |  83.3+/-0.6  | 71.8+/-0.4    |  80.2+/-0.2   
simple-NAS |  81.4+/-0.6  |  71.7+/-0.6    |  79.5+/-0.5  
GraphNAS | **84.3+/-0.4**  | **73.7+/-0.2**    | **80.6+/-0.2**  
		
Supervised node classification w.r.t. accuracy	

Model| Cora | Citeseer | Pubmed  
|-|-|-|-| 
GCN    | 90.2+/-0.0  | 80.0+/-0.3   | 87.8+/-0.2  
SGC    | 88.8+/-0.0 |  80.6+/-0.0   |   86.5+/-0.1  
GAT    |  89.5+/-0.3  |  78.6+/-0.3    |  86.5+/-0.6   
LGCN    | 88.7+/-0.5  | 79.2+/-0.4     |  OOM    
DGCN    |  88.4+/-0.2  |  78.0+/-0.2    |  88.0+/-0.9    
ARMA    |  89.8+/-0.1  |  79.9+/-0.6    |  88.1+/-0.2    
APPNP    | 90.4+/-0.2  | 79.2+/-0.4     | 87.4+/-0.3    
random-NAS | 90.0+/-0.3   |  81.1+/-0.3    | 90.7+/-0.6    
simple-NAS | 90.1+/-0.3  |  79.6+/-0.5    |  88.5+/-0.2  
GraphNAS | **90.6+/-0.3**   |  **81.3+/-0.4**   | **91.3+/-0.3**    
	
Architectures designed in supervised learning are showed as follow:
<p align="center">
<img src="./images/GraphNAS_cells.png" width="500"  alt="Architectures designed by GraphNAS in supervised experiments" align=center>
</p>
The architecture G-Cora designed by GraphNAS on Cora is [0, gat6, 0, gcn, 0, gcn, 2, arma, tanh, concat], 
the architecture G-Citeseer designed by GraphNAS on  Citeseer is [0, identity, 0, gat6, linear, concat], 
the architecture G-Pubmed designed by GraphNAS on  Pubmed is [1, gat8, 0, arma, tanh, concat]. 

##### Searching for new architectures
To design an entire graph neural architecture based on the search space described in Section 3.2, please run: 

    python -m graphnas.main --dataset Citeseer

To design an entire graph neural architecture based on the search space described in Section 3.4, please run: 
    
    python -m graphnas.main --dataset Citeseer --supervised True --search_mode micro

Be aware that different runs would end up with different local minimum.

#### Acknowledgements
This repo is modified based on [DGL](https://github.com/dmlc/dgl) and [PYG](https://github.com/rusty1s/pytorch_geometric).
