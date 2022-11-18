## 計算の進め方
### インプットファイル(in.dat)
- ファイル名など

DATA.PATHには擬ポテンシャル関数が格納されたディレクトリを指定すること、ディレクトリの階層に注意。

```
#
#      File Name      
#

System.CurrrentDirectory         ./    # default=./
System.Name                      result
level.of.stdout                   1    # default=1 (1-3)
level.of.fileout                  1    # default=1 (1-3)
DATA.PATH			../DFT_DATA19 
```

- 原子種の定義

Species.Numberには計算で取り扱う原子種の数を入力する。

Definiction.of.Atomic.Speciesには以下を入力する。

(1) 原子記号

(2) Pseudo-atomic orbitals (Cutoff半径): 
OpenMXのHPの[Database](https://www.openmx-square.org/vps_pao2019/)を確認し、"Calculation of the total energy as a function of lattice constant in the diamond structure"のグラフの中で、比較的一致している曲線の中から電子数の少ない軌道を選ぶとよい。

(3) Fully relativistic pseudopotentials:
同じくHPのDatabaseを見て、「*_PBE*」を選択する

```
#
# Definition of Atomic Species
#

Species.Number       1
<Definition.of.Atomic.Species
   Cu   Cu6.0S-s2p2d2   Cu_PBE19S
Definition.of.Atomic.Species>
```


- 各原子の情報

Atoms.Numberには取り扱う原子の個数を入力する。

Atoms.SpeciesAndCoordinates.Unitには、絶対座標Angか、相対座標FRAC(計算ボックスの各ベクトルの長さを1としたときの比率)を選ぶ。

Atoms.SpeciesAndCoordinatesには、各原子の座標(v1, v2, v3)と、価電子の数(n1, n2 ※合わせて総価電子数となるように)を指定する。
※価電子の数は先のDatabaseを見れば書いてある。

Atoms.UnitVectorsには、計算BOXを構成する単位ベクトルを記載する(これで計算ボックスサイズが決まる)。

```
#
# Atoms
#

Atoms.Number         32
Atoms.SpeciesAndCoordinates.Unit   FRAC # Ang|AU
<Atoms.SpeciesAndCoordinates           # Unit=Ang.
	1	Cu	0.00	0.00	0.00	6.0		5.0
	2	Cu	0.25	0.25	0.00	6.0		5.0
	3	Cu	0.25	0.00	0.25	6.0		5.0
	4	Cu	0.00	0.25	0.25	6.0		5.0
	5	Cu	0.50	0.00	0.00	6.0		5.0
	6	Cu	0.75	0.25	0.00	6.0		5.0
	7	Cu	0.75	0.00	0.25	6.0		5.0
	8	Cu	0.50	0.25	0.25	6.0		5.0
	9	Cu	0.00	0.50	0.00	6.0		5.0
	10	Cu	0.25	0.75	0.00	6.0		5.0
	11	Cu	0.25	0.50	0.25	6.0		5.0
	12	Cu	0.00	0.75	0.25	6.0		5.0
	13	Cu	0.00	0.00	0.50	6.0		5.0
	14	Cu	0.25	0.25	0.50	6.0		5.0
	15	Cu	0.25	0.00	0.75	6.0		5.0
	16	Cu	0.00	0.25	0.75	6.0		5.0
	17	Cu	0.50	0.50	0.00	6.0		5.0
	18	Cu	0.75	0.75	0.00	6.0		5.0
	19	Cu	0.75	0.50	0.25	6.0		5.0
	20	Cu	0.50	0.75	0.25	6.0		5.0
	21	Cu	0.50	0.00	0.50	6.0		5.0
	22	Cu	0.75	0.25	0.50	6.0		5.0
	23	Cu	0.75	0.00	0.75	6.0		5.0
	24	Cu	0.50	0.25	0.75	6.0		5.0
	25	Cu	0.00	0.50	0.50	6.0		5.0
	26	Cu	0.25	0.75	0.50	6.0		5.0
	27	Cu	0.25	0.50	0.75	6.0		5.0
	28	Cu	0.00	0.75	0.75	6.0		5.0
	29	Cu	0.50	0.50	0.50	6.0		5.0
	30	Cu	0.75	0.75	0.50	6.0		5.0
	31	Cu	0.75	0.50	0.75	6.0		5.0
	32	Cu	0.50	0.75	0.75	6.0		5.0
Atoms.SpeciesAndCoordinates>
Atoms.UnitVectors.Unit             Ang #  Ang|AU
<Atoms.UnitVectors                     # unit=Ang.
  7.229500000000  0.000000000000  0.000000000000
  0.000000000000  7.229500000000  0.000000000000
  0.000000000000  0.000000000000  7.229500000000
Atoms.UnitVectors>
```

- 計算条件

```
#
# SCF or Electronic System
#

scf.XcType                 GGA-PBE     # LDA|LSDA-CA|LSDA-PW
scf.SpinPolarization       off         # On|Off
scf.ElectronicTemperature  300.0       # default=300 (K)
scf.energycutoff           160.0       # default=150 (Ry)
scf.maxIter                100         # default=40
scf.EigenvalueSolver       band        # Recursion|Cluster|Band
scf.lapack.dste            dstevx      # dstegr|dstedc|dstevx, default=dstegr
scf.Kgrid                  2 2 2       # means 4x4x4
scf.Mixing.Type           rmm-diisk    # Simple|Rmm-Diis|Gr-Pulay
scf.Init.Mixing.Weight     0.010       # default=0.30 
scf.Min.Mixing.Weight      0.001       # default=0.001 
scf.Max.Mixing.Weight      0.200       # default=0.40 
scf.Mixing.History         15          # default=5
scf.Mixing.StartPulay       5          # default=6
scf.criterion             1.0e-7       # default=1.0e-6 (Hartree) 
```

- MD計算条件

```
#
# MD or Geometry Optimization
#

MD.Type                     Nomd        # Nomd|Opt|DIIS|NVE|NVT_VS|NVT_NH
MD.Opt.DIIS.History          7         # default=7
MD.Opt.StartDIIS             5         # default=5
MD.maxIter                  100        # default=1
MD.TimeStep                1.0         # default=0.5 (fs)
MD.Opt.criterion          1.0e-4       # default=1.0e-4 (Hartree/bohr)
```
