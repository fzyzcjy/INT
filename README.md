# INT

This is the official code for the paper
[INT: An Inequality Benchmark for Evaluating Generalization in Theorem Proving](https://arxiv.org/abs/2007.02924).

To cite the paper, use:

```
@inproceedings{WJBG2021INT,
  author    = {Yuhuai Wu and
               Albert Jiang and
               Jimmy Ba and
               Roger Baker Grosse},
  title     = {{INT:} An Inequality Benchmark for Evaluating Generalization in Theorem
               Proving},
  booktitle = {9th International Conference on Learning Representations, {ICLR} 2021,
               Virtual Event, Austria, May 3-7, 2021},
  publisher = {OpenReview.net},
  year      = {2021},
  url       = {https://openreview.net/forum?id=O6LPudowNQm},
}
```

This directory contains

* a light-weight theorem prover with fast interactions
* the generator of theorems and proofs in the INT dataset
* the scripts for training agents to prove such theorems

## Installation

The software packages required are:

[PyTorch](https://pytorch.org/)

[PyTorch geometric](https://pytorch-geometric.readthedocs.io/en/latest/)

[DGL](https://docs.dgl.ai/)

[pytorch-a2c-ppo-acktr](https://github.com/ikostrikov/pytorch-a2c-ppo-acktr-gail)

[OpenAI Gym](https://gym.openai.com/)

[OpenAI Baselines](https://github.com/openai/baselines)

## Setup

```bash
git clone https://github.com/albertqjiang/INT
cd INT
```

## Data

To generate the theorems and proofs, we need to first generate the axiom combinations and orders to use.
The following command generates the axiom combinations and orders
for fewer than 5 unique axioms and 5 axiom applications in a proof, with 10000 trials for each (k, l) pair.

```bash
python -m int_environment.data_generation.combos_and_orders --combo_path data/benchmark/field --max_k 5 --max_l 5 --trial 10000
```

After getting the axiom combinations and orders, we can use them to generate synthetic theorems and proofs.
The command below generates 100 separate theorems and corresponding proofs, for graph processing,
with the parameters k=5, l=5.

```bash
python -m int_environment.data_generation.generate_problems --orders_path data/benchmark/field -k 5 -l 5 --num_probs 100 -dp $dump_path
```

For seq2seq training, use the following command instead:

```bash
python -m int_environment.data_generation.gen_seq2seq --orders_path data/benchmark/field/  -k 5 -l 5 --num_probs 100 -dp $dump_path
```

To visualise 5 proofs just generated using the seq2seq mode, use the following command:

```bash
python -m int_environment.visualization.display_proofs --proof-file $dump_path/problems.pkl --how-many 5
```

## Training

We prepare the training script for a graph-neural-network based agent.
The command below trains the agent for 1 million updates in an online setting.
We generate 1000 theorems and proofs with the setting k=5, l=5
and train 10 epochs on them before generating a new set of problems.
The training results and models are dumped in directory `data/pt_models`.

```bash
mkdir data/pt_models
python -m int_environment.algos.main -cp data/benchmark/field -du data/pt_models/ --online -trs k\=5_l\=5 -tes k\=5_l\=5 -epod 10 -np 1000 --lr 1e-4 -u 1000000 -tg --degree 0 --seed 0
```

## Executing tests

Simply run:

```
pytest test
```

## Documentation for important functions and classes

[INT/data_generation/generate_problems.py:268](https://github.com/albertqjiang/INT/blob/2ec739c94b2feb5f7f80b3d5e71e8b751dbd9ef3/data_generation/generate_problems.py#L268)

```python
def generate_multiple_problems(num_axioms, length, num_probs, **kwargs):
    """
    Generate multiple theorems and proofs and return the tuple (datasets, problems)

    :param num_axioms: the number of unique axioms in the proofs generated

    :param length: the length of the proofs generated

    :param num_probs: how many theorems and proofs to generate

    :param kwargs: keyword arguments to provide additional specifications for the problems generated

        :keyword train_test: str, optional
        Could be "train" or "test", specifies which partition of the axiom combinations or orders to use, default: "train"

        :keyword combos: dict, optional 
        The dictionary containing all the available combinations to use in generation, cannot appear at the same time as orders
        
        :keyword orders: dict, optional
        The dictionary containing all the available orders to use in generation, cannot appear at the same time as combos
        
        :keyword avoid_objective_names: list, optional
        The list containing all the objective names we wish to avoid during generation. Use this to prevent having test problems, default: []
        
        :keyword ivs: list, optional
        Individual variables to initialize the prover, default: [a, b, c]
        
        :keyword ed: dict, optional
        Entity dictionary of the initial prover, default: [0:a, 1:b, 2:c]
        
        :keyword ent_per_degree: int, optional
        The number of entities to sample from each degree, default: 10
        
        :keyword degree: int, optional
        The maximum degree of entities sampled, default: 0
        
        :keyword no_atom_ent_max: int, optional
        Maximum number of atomic entities, default: 20
        
        :keyword no_node_max: int, optional
        Maximum number of nodes in the graphs, default: 2000
        
        :keyword entity_length_limit: int, optional
        Maximum length of entities in characters, default: 20
        
        :keyword transform_gt: bool, optional
        Whether to enable transform_gt, default: True
        
        :keyword probability_no_transform: float, optional
        Probability of not doing transform_gt in each step, default 0.0

    :return: tuple (Datasets, Problems)

        Datasets is a list containing:
            One dataset with the keyword "all" contains all the proof steps generated, randomly shuffled
            One dataset with the keyword "all_first" contains the first proof steps of each problem, randomly shuffled
        
        Problems is a list, each element of which is all the proof steps for an individual theorem
    """
```

[INT/proof_system/prover.py:4](https://github.com/albertqjiang/INT/blob/2ec739c94b2feb5f7f80b3d5e71e8b751dbd9ef3/proof_system/prover.py#L4)

```python
class Prover:
    def __init__(self, axioms: dict, conditions: list, objectives: list, prove_direction: str):
        """
        This prover is capable of doing forward generation and backward proving
        :param axioms: the axioms that can be used inside the prover
        :param conditions: the conditions to start the proof with
        :param objectives: the objectives to prove, usually only one
        :param prove_direction: either "forward" or "backward",
                                use forward for generation and backward for writing proofs
        """

    @staticmethod
    def _trivial(logic_statement):
        """
        Checks if a logic statement is trivially true or not
        """

    def update_conditions(self):
        """
        Register the initial conditions as ground truth
        """

    def add_logic_statement(self, logic_statement):
        """
        Index and register a logic statement
        """

    def add_logic_statements(self, logic_statement_list):
        """
        Index and register multiple logic statements
        """

    def interpret_result(self, result):
        """
        Interpret the result of a theorem application
        """

    def _parse_entity_ids_from_entity(self, entity):
        """
        Get the ids of all entities that are subtrees of a given entity(including itself)
        """

    def _parse_entity_ids_from_ls(self, logic_statement):
        """
        Get the ids of all entities in a logic statement
        """

    def _add_entity(self, entity):
        """
        Add one entity to the register, return the id.
        """

    def _add_entities(self, entities):
        """
        Add multiple entities to the register, return a list of the ids of the entities
        """

    def get_entities(self):
        """
        Get all the entities in the prover ever registered
        """

    def get_ground_truth(self):
        """
        Get all the ground truth in the prover ever registered,
        including the initial conditions and statements proven later
        """

    def get_objectives(self):
        """
        Get the current objectives in the prover
        """

    def get_observation(self):
        """
        Get the observation for the prover environment, this should contain everything needed to prove the theorem
        """

    def logic_statement_connected(self, ls_id):
        """
        Determine whether the logic statement with the id is connected to premises
        """

    def _logic_statements_exist_and_are_proven(self, ls_list):
        """
        Determine if all logic statements in ls_list exist in the prover and are proven
        """

    def apply_theorem(self, theorem, operands):
        """
        Apply a theorem with operands
        """

    def is_proved(self):
        """
        Determine whether the proof is complete
        """
```
