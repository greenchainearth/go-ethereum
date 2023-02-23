**Go Ethereum over seed**
Note

Repo is the github.com/ethereum/go-ethereum fork. It should be in the same local path as original: $GOPATH/src/github.com/ethereum/.
Aims

Ethereum over seed network and consensus. Full ethereum stack and seed performance.
Demo

    start seed test net first: cd $GOPATH/src/github.com/greenchainearth/go-forest/demo && make start;
    change dir: cd seed/demo;
    make geth docker image and run 2 containers: make && make start;
    try to send tx: ./20.txn.sh;
    try to get balanses: ./10.balances.sh;
    stop docker containers: make stop;
    stop seed test net;

Changes

    Rename p2p.Server to p2p.p2pServer;
    Create p2p/interface.go (p2p.ServerInterface, p2p.Server struct, p2p.p2pServer's additionals methods, p2p.seedAdapter interface);
    Create eth/seed.go (p2p.SeedAdapter implementation);
    Add p2p.Config.SeedAdapter SeedAdapter;
    Create SeedAddrFlag in cmd/utils/flags.go:

        SeedAddrFlag = cli.StringFlag{
    	Name:  "seed",
    	Usage: "seed-node address",
    }
    . . .
    func setListenAddress(ctx *cli.Context, cfg *p2p.Config) {
    	. . .
    	if ctx.GlobalIsSet(SeedAddrFlag.Name) {
    		cfg.SeedAdapter = eth.NewSeedAdapter(ctx.GlobalString(SeedAddrFlag.Name), cfg.Logger)
    	}
    }

    Append utils.SeedAddrFlag to:
        nodeFlags in cmd/geth/main.go;
        AppHelpFlagGroups in cmd/geth/usage.go;
        app.Flags in cmd/swarm/main.go;
    Make node.Node create .server according to .serverConfig.SeedAdapter and use p2p.Server.AddProtocols() at .Start():

    var running *p2p.Server
    if n.serverConfig.SeedAdapter == nil {
    	running = p2p.NewServer(n.serverConfig)
    	n.log.Info("Starting peer-to-peer node", "instance", n.serverConfig.Name)
    } else {
    	running = seed.NewServer(n.serverConfig)
    	n.log.Info("Using seed node", "address", n.serverConfig.SeedAdapter.Address())
    }
    . . .
    for _, service := range services {
    	running.AddProtocols(service.Protocols()...)
    }

    Create seed/ package;

TODO:

    make ethereum blocks from seed commits at eth.seedAdapter without mining;
    switch seed/demo/docker/Dockerfile.geth* from local to origin "github.com/greenchainearth/go-forest" when stable;
