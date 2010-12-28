cinc   = '-Isrc/inc'
cflag  = "-Wall -finline-functions -nostdinc -fno-builtin -fno-stack-protector"
pgrep  = "grep --color -e 'error' -e 'error' -e '^'"

task :default => :bochs

task :bochs => :build do
  sh "bochs -q -f .bochsrc"
end

task :nm => :build do 
  sh 'cat main.nmtab'
end

task :debug => :build do
  sh "bochs-dbg -q -f .bochsrc"
end
  
task :build => ['bin/kernel.img', :rootfs, :ctags]

task :clean do
  sh "rm -rf bin/* src/kern/entry.S .bochsout"
end

## helpers ##
task :todo do 
  sh "grep -r -n '\\(TODO\\|\\<bug\\>\\)' ./src --color"
end

task :ctags do
  sh "ctags -R"
end

# 
# the root file system aka the hard disk image, 1mb yet and ignored
# partition
# note: for the mounting, a root privilege is required.
# 
task :rootfs => ['bin/rootfs.img']

desc 'init and copy some thing into the hard disk image.'
file 'bin/rootfs.img' do 
  sh "bximage bin/rootfs.img -hd -mode=flat -size=1 -q"
  sh "mkfs.minix bin/rootfs.img"
  mkdir_p '/tmp/fx_mnt_root'
  `sudo umount /tmp/fx_mnt_root`
  sh "sudo mount -o loop -t minix bin/rootfs.img /tmp/fx_mnt_root"
  sh "cp -r ./root/* /tmp/fx_mnt_root"
  sh "sudo umount /tmp/fx_mnt_root"
  sh "rm -rf /tmp/fx_mnt_root"
end

# 
# the kernel image, a concat of boot image and the main binary.
# 
file 'bin/kernel.img' => ['bin/boot.bin', 'bin/main.bin'] do
  sh "cat bin/boot.bin bin/main.bin > bin/kernel.img"
end

# 
# boot.o
#
file 'bin/boot.o' => ['src/boot/boot.S'] do
  sh "nasm -f elf -o bin/boot.o src/boot/boot.S"
end

file 'bin/boot.bin' => ['bin/boot.o', 'boot.ld'] do 
  sh "ld bin/boot.o -o bin/boot.bin -e c -T boot.ld"
end

# ---------------------------------------------------------------------
# mainly C part
# => main.bin
# ---------------------------------------------------------------------

hfiles = Dir['src/inc/*.h']

cfiles = [
  'src/kern/tty.c',
  'src/kern/sys.c',
  'src/kern/sched.c',
  'src/kern/seg.c',
  'src/kern/trap.c',
  'src/kern/timer.c',
  #
  'src/mm/page.c',
  #
  'src/drv/buf.c',
  'src/drv/conf.c',
  'src/drv/hd.c',
  #
  'src/fs/super.c',
  'src/fs/inode.c',
  'src/fs/mount.c',
  'src/fs/namei.c',
  'src/fs/rdwri.c',
  'src/fs/file.c',
  'src/fs/alloc.c',
  'src/fs/bmap.c',
  #
  'src/lib/string.c',
  'src/lib/bitmap.c',
  'src/lib/unistd.c',
  #
  'src/kern/main.c'
]

sfiles = [
  'src/kern/entry.S'
]

ofiles = (cfiles + sfiles).map{|fn| 'bin/'+File.basename(fn).ext('o') }

cfiles.each do |fn_c|
  fn_o = 'bin/'+File.basename(fn_c).ext('o')
  file fn_o => [fn_c, *hfiles] do
    sh "gcc #{cflag} #{cinc} -o #{fn_o} -c #{fn_c} 2>&1"
  end
end

sfiles.each do |fn_s|
  fn_o = 'bin/'+File.basename(fn_s).ext('o')
  file fn_o => [fn_s, *hfiles] do
    sh "nasm -f elf -o #{fn_o} #{fn_s}"
  end
end

# ---------------------------------------------------------------------3

file 'bin/main.bin' => 'bin/main.elf' do
  sh "objcopy -R .pdr -R .comment -R .note -S -O binary bin/main.elf bin/main.bin"
end

file 'bin/main.elf' => ofiles + ['main.ld'] do
  sh "ld #{ofiles * ' '} -o bin/main.elf -e c -T main.ld"
  sh "(nm bin/main.elf | sort) > main.sym"
end

# entry.S is generated by a ruby script
file 'src/kern/entry.S' => 'src/kern/entry.S.rb' do 
  sh 'ruby src/kern/entry.S.rb > src/kern/entry.S'
end


