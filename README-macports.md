	sudo port install gcc48 +universal gcc49 +universal gcc5 +universal gcc6 +universal
	
	mkdir build-gcc48
	cd build-gcc48
	cmake .. -DGCC_VERSION=4.8.5 -DIBERTY_LIB=/opt/local/lib/gcc48/libiberty.a
	make
	cd ..
	
	mkdir build-gcc49
	cd build-gcc49
	cmake .. -DGCC_VERSION=4.9.4 -DIBERTY_LIB=/opt/local/lib/gcc48/libiberty.a
	make
	cd ..
	
	mkdir build-gcc5
	cd build-gcc5
	cmake .. -DGCC_VERSION=5.4.0 -DIBERTY_LIB=/opt/local/lib/gcc48/libiberty.a
	make
	cd ..
	
	mkdir build-gcc6
	cd build-gcc6
	cmake .. -DGCC_VERSION=6.2.0 -DIBERTY_LIB=/opt/local/lib/gcc48/libiberty.a
	make
	cd ..
