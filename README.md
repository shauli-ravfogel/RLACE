### Background

This repository contains an implementation of Relaxed Linear Adversarial Concept Erasure (RLACE). Given a dataset `X` of dense representations and labels `y` for some concept (e.g. gender), the method identifies a rank-`k` subsapce whose neutralization (suing an othogonal projection matrix) prevents linear classifiers from recovering the concept from the representations. 

The method relies on a relaxed and constrained version of a minimax game between a predictor that aims to predict `y` and a projection matrix `P` that is optimized to prevent the prediction.

### How to run
A simple running example is provided withint `rlace.py`.

#### Parameters
The main method, `solve_adv_game`, receives several arguments, among them:

`rank`: the rank of the neutralized subspace. `rank=1` is emperically enough to prevent linear prediction in binary classification problem.

`epsilon`: stopping criterion for the adversarial game. Stops if abs(acc - majority_acc) < epsilon.

`optimizer_class`: torch.optim optimizer

`optimizer_params_predictor / optimizer_params_P`: parameters for the optimziers of the predictor and the projection matrix, respectively.


#### Running example:

```
num_iters = 50000
rank=1
optimizer_class = torch.optim.SGD
optimizer_params_P = {"lr": 0.003, "weight_decay": 1e-4}
optimizer_params_predictor = {"lr": 0.003,"weight_decay": 1e-4}
epsilon = 0.001 # stop 0.1% from majority acc
batch_size = 256

output = solve_adv_game(X_train, y_train, X_dev, y_dev, rank=rank, device="cpu", out_iters=num_iters, optimizer_class=optimizer_class, optimizer_params_P =optimizer_params_P, optimizer_params_predictor=optimizer_params_predictor, epsilon=epsilon,batch_size=batch_size)
```

**Optimization**: Even though we run a concave-convex minimax game, which is generallly "well-behaved", optimziation with alternate SGD is still not completely straightforward, and may require some tuning of the optimizers. Accuracy is also not expected to monotonously decrease in optimization; we return the projection matrix which performed best along the entire game. In all experiments on binary classification problems, we identified a projection matrix that neutralizes a rank-1 subspace and decreases classification accuracy to near-random (50%).

#### Usage:


`output` that is returned from `solve_adv_game` is a dictionary, that contains the following keys:

`score`: final accuracy of the predictor on the projected data.

`P_before_svd`: the final approximate projection matrix, before SVD that guarantees it's a proper orthogonal projection matrix.

`P`: a proper orthogonal matrix that neutralizes a rank-`k` subspace. 

**Using the projection**: After the adversarial game is terminated, the ``clean" vectors are given by `X.dot(output["P"])`.
