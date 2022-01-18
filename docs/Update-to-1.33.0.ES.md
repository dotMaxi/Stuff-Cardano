Esta guia pretende dar cobertura a la actualización del nodo de Cardano a la versión 1.33.0. Son muchos los que usaban la guia de [Coincashew](https://www.coincashew.com/coins/overview-ada/guide-how-to-build-a-haskell-stakepool-node/how-to-update-a-stakepool) pero la última vez que la actualizaron fue para la versión 1.30.1 el 1 de octubre del 2021.  
  
Aunque practicamente en cada versión siempre se realizan las mismas acciones considero que puede llegar a ser útil para algunos usuarios. De hecho, en esta actualización, a diferencia de las anteriores, vamos a usar una nueva versión de GHC.  
  
**1. Consideraciones antes de actualizar el nodo de Cardano**  
- Realiza un snapshot del servidor si tienes la posibilidad de hacerlo.  
- Actualiza primero el nodo de un relay.
- Ten en cuenta que en esta versión, una vez finalizada la actualización puede demorarse en arrancar el nodo hasta 4 horas o incluso más dependiendo del hardware del servidor.

**2. Actualización del sistema**  
  
Antes de continuar, actualizamos el sistema y reiniciamos el servidor.  
```bash
sudo apt-get update -y && sudo apt-get upgrade -y && sudo reboot
```  
**3. Descarga de la última versión del nodo de Cardano**  
```bash
cd $HOME/git
git clone https://github.com/input-output-hk/cardano-node.git cardano-node2
cd cardano-node2/
git fetch --all --recurse-submodules --tags
git checkout tags/1.33.0
```  
Hasta el momento, en versiones anteriores del nodo de Cardano se estaba usando GHC 8.10.4. A partir de la 1.33.0 
usaremos la versión 8.10.7 de acuerdo a las instrucciones de IOK en su github: https://github.com/input-output-hk/cardano-node/blob/master/doc/getting-started/install.md  
```bash
ghcup upgrade
ghcup install ghc 8.10.7
ghcup set ghc 8.10.7
ghcup install cabal 3.4.0.0
ghcup set cabal 3.4.0.0
cabal update
ghc --version
cabal --version
```  
**4. Construyendo el nodo**  
```bash
cd $HOME/git/cardano-node2
cabal configure -O0 -w ghc-8.10.4
echo -e "package cardano-crypto-praos\n flags: -external-libsodium-vrf" > cabal.project.local
cabal build cardano-node cardano-cli
```  

**Pendiente de completar guia.....**
