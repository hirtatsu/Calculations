## 計算の進め方
### インプットファイル(in.dat)
- ファイル名など
```
#
#      File Name      
#

System.CurrrentDirectory         ./    # default=./
System.Name                      result
level.of.stdout                   1    # default=1 (1-3)
level.of.fileout                  1    # default=1 (1-3)
DATA.PATH			../DFT_DATA19 # 擬ポテンシャル関数が格納されたディレクトリを指定すること、ディレクトリの階層に注意。
```
- 原子種の定義
```
#
# Definition of Atomic Species
#

Species.Number       1 # 元素の種類数
<Definition.of.Atomic.Species
   Cu   Cu6.0S-s2p2d2   Cu_PBE19S
Definition.of.Atomic.Species>
```
(1) 原子記号, (2) Pseudo-atomic orbitals (Cutoff半径), (3) Fully relativistic pseudopotentials。
OpenMXのHPの[Database](https://www.openmx-square.org/vps_pao2019/)を確認する。


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