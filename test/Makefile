
all: testMain_opt.o
	g++ -O0 -g --std=c++17 testMain_opt.o tests.C -fmax-errors=1 -o tests_opt.exec

testMain_opt.o: testMain.C
	g++ -O3 -g --std=c++17 testMain.C -c -o testMain_opt.o
