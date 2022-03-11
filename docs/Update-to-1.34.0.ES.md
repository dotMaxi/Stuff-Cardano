En construcción...

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
git checkout tags/1.34.0
```  
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
echo -e "package cardano-crypto-praos\n flags: -external-libsodium-vrf" > cabal.project.local
cabal build cardano-node cardano-cli
```  
**5. Comprobando la versión de cardano-cli y cardano-node**  
```bash
$(find $HOME/git/cardano-node2/dist-newstyle/build -type f -name "cardano-cli") version
$(find $HOME/git/cardano-node2/dist-newstyle/build -type f -name "cardano-node") version
```  
**6. Apagando el nodo y copiando la nueva versión**  
```bash
sudo systemctl stop cardano-node
```  
Ahora copiamos cardano-cli y cardano-node en /usr/local/bin/ con el siguiente comando:  
```bash
sudo cp $(find $HOME/git/cardano-node2/dist-newstyle/build -type f -name "cardano-cli") /usr/local/bin/cardano-cli
sudo cp $(find $HOME/git/cardano-node2/dist-newstyle/build -type f -name "cardano-node") /usr/local/bin/cardano-node
```  
**7. Comprobando la versión del nodo de Cardano que se va a ejecutar**  
```bash
cardano-node version
cardano-cli version
```  
Arrancamos el nodo de Cardano:
```bash
sudo systemctl start cardano-node
```  
**8. Limpiando** 
```bash
cd $HOME/git/
rm -rf cardano-node-old
mv cardano-node cardano-node-old
mv cardano-node2 cardano-node
```
**9. Revisando logs** 
```bash
journalctl --unit=cardano-node --follow 
```
