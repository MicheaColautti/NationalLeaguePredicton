## Comandi da eseguire per installare le libreire necessarie
```sh
conda create -n sports_sihf python=3.12 pip -y

conda activate sports_sihf

python -m pip install --upgrade pip setuptools wheel

python -m pip install --upgrade "sportsdataverse==0.2.0b1"

python -m pip install pandas numpy scipy pyarrow matplotlib seaborn scikit-learn requests tqdm jupyterlab notebook ipykernel

python -m ipykernel install --user --name sports_sihf --display-name "Python (sports_sihf)"

python -c "import sys; from importlib.metadata import version; from sportsdataverse.hockey.sihf import sihf_results, sihf_game_team_stats, sihf_game_player_stats; print('Python:', sys.executable); print('SportsDataverse:', version('sportsdataverse')); print('Import SIHF riuscito!')"
```
