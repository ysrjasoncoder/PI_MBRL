# PI_MBRL
MBRL with a Trained Physics-Informed Model

## Task 1: MBRL with a Trained Physics-Informed Model

Description: The first task serves as a baseline. Instead of learning a dynamics model from scratch as in the provided algorithm, we will assume access to a high-fidelity, but potentially imperfect, physics-informed model, $\hat{f}_\text{phys}$. This model is developed and trained offline and is not updated during the agent's interaction with the environment. This approach completely removes the "model-learning" part from the online loop, turning the algorithm into a pure Model Predictive Control (MPC) problem where the pre-built model is used for planning.

Algorithm 1: Model Predictive Control with a Static Physics-Informed Model

```
1:  # Offline Phase
2:  Develop and/or train a physics-informed model of the system dynamics, denoted as \\hat{f}_{phys}(s, a).
3:  Load the finalized, trained physics-informed model \hat{f}_{phys}.
4:
5:  # Online Execution/Control Phase
6:  for t = 1 to T do
7:      get agent's current state s_t
8:      use the static model \hat{f}_{phys} to estimate the optimal action sequence A_t^{(H)} by solving a trajectory optimization problem.
9:      execute the first action a_t from the selected action sequence A_t^{(H)}.
10:     observe the resulting state s_{t+1}.
11: end for
```

## Task 2: MBRL with a Hybrid Physics-Residual Model

Description: This task addresses the common scenario where the physics-informed model $\hat{f}\text{phys}$ is only an approximation of the true dynamics. To account for the model mismatch (the "sim-to-real gap"), we introduce a learnable correction term, $\hat{f}\theta$. The full dynamics model is a hybrid composition: $\hat{f}\text{hybrid}(s,a)=\hat{f}\text{phys}(s,a)+\hat{f}\theta(s,a)$. Here, $\hat{f}\theta$ is an affine correction model (e.g., a neural network) trained online to predict the residual or error between the physics model's predictions and the observed reality. This approach follows the structure of the original algorithm but reframes the learning problem from learning the full dynamics to learning only the correction term, which is often a simpler task.

Algorithm 2: Hybrid Physics-Residual MBRL

```
1:  Load the approximate physics-informed model \\hat{f}_{phys}.
2:  gather initial dataset \\mathcal{D}_{RAND} of random trajectories (s, a, s').
3:  initialize empty dataset \\mathcal{D}_{RL}, and randomly initialize the correction model \\hat{f}_{\\theta}.
4:
5:  for iter = 1 to max_iter do
6:      # Train the correction model to predict the residual
7:      train \\hat{f}_{\\theta}(s, a) by performing gradient descent to minimize the loss || (s' - \\hat{f}_{phys}(s, a)) - \\hat{f}_{\\theta}(s, a) ||^2 for transitions (s, a, s') in \\mathcal{D}_{RAND} and \\mathcal{D}_{RL}.
8:
9:      # Collect more data using the hybrid model
10:     for t = 1 to T do
11:         get agent's current state s_t.
12:         define the full hybrid model: \\hat{f}_{hybrid}(s, a) = \\hat{f}_{phys}(s, a) + \\hat{f}_{\\theta}(s, a).
13:         use \\hat{f}_{hybrid} to estimate the optimal action sequence A_t^{(H)}.
14:         execute the first action a_t from A_t^{(H)} and observe the next state s_{t+1}.
15:         add (s_t, a_t, s_{t+1}) to \\mathcal{D}_{RL}.
16:     end for
17: end for
```

## Task 3: Uncertainty-Aware MBRL with a Probabilistic Residual Ensemble

Description: The final task extends the hybrid model by incorporating uncertainty quantification. Instead of learning a single correction model $\hat{f}\theta$, we will train an ensemble of $N$ correction models, $\{\hat{f}{\theta_i}\}_{i=1}^N$. Each model in the ensemble is trained on a different subset of the collected data (using bootstrapping). The disagreement between the predictions of these models serves as a proxy for epistemic uncertainty. This uncertainty can then be leveraged during the planning phase to encourage exploration in state-action regions where the model is uncertain and to favor safer trajectories. This is a common technique in state-of-the-art probabilistic MBRL.

Algorithm 3: Probabilistic Hybrid Physics-Residual MBRL

```
1:  Load the approximate physics-informed model \\hat{f}_{phys}.
2:  gather initial dataset \\mathcal{D}_{RAND} of random trajectories (s, a, s').
3:  initialize empty dataset \\mathcal{D}_{RL}, and randomly initialize an ensemble of N correction models \\{\\hat{f}_{\\theta_i}\\}_{i=1}^N.
4:
5:  for iter = 1 to max_iter do
6:      # Train the ensemble of correction models
7:      for i = 1 to N do
8:          Create a bootstrap sample \\mathcal{D}_{boot, i} from the full dataset (\\mathcal{D}_{RAND} \\cup \\mathcal{D}_{RL}).
9:          Train \\hat{f}_{\\theta_i}(s, a) to predict the residual on \\mathcal{D}_{boot, i}.
10:     end for
11:
12:     # Collect more data using uncertainty-aware planning
13:     for t = 1 to T do
14:         get agent's current state s_t.
15:
16:         # Uncertainty-aware planning (e.g., using trajectory sampling)
17:         Use the ensemble \\{\\hat{f}_{\\theta_i}\\}_{i=1}^N to estimate an optimal action sequence A_t^{(H)}. Planning incorporates uncertainty, for example, by propagating particles through different models from the ensemble and selecting the action sequence that maximizes expected reward.
18:
19:         execute the first action a_t from A_t^{(H)} and observe the next state s_{t+1}.
20:         add (s_t, a_t, s_{t+1}) to \\mathcal{D}_{RL}.
21:     end
```