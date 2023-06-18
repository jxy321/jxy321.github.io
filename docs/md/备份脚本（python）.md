# 备份脚本（python）

```python
'''
1、既要可以实现完全备份，又要实现增量备份
2、完全备份时，将目录打个tar包，计算每个文件的md5值
3、增量备份时，备份有变化的文件和新增加的文件，更新md5值
'''
import time
import os
import tarfile
import hashlib
import pickle

###########################记录十六进制md5值#############################
def check_md5(fname):
  m = hashlib.md5()
  with open(fname, 'rb') as fobj:
    while True:
      data = fobj.read(4096)
      if not data:
        break
      m.update(data)
  return m.hexdigest()
#############################完整备份###################################
def full_backup(src_dir, dst_dir, md5file):
  #记录文件名
  fname = os.path.basename(src_dir.rstrip('/'))
  #修改tar包名
  fname = '%s_full_%s.tar.gz' % (fname, time.strftime('%Y%m%d'))
  #记录文件路径名
  fname = os.path.join(dst_dir, fname)
  md5dict = {}
  #制作tar包
  tar = tarfile.open(fname, 'w:gz')
  tar.add(src_dir)
  tar.close()
  #遍历源目录里的文件，记录每一个md5值，与文件一一对应写入md5dict字典中
  for path, folders, files in os.walk(src_dir):
    for each_file in files:
      key = os.path.join(path, each_file)
      md5dict[key] = check_md5(key)
  #最后将md5dict字典以二进制格式写入文件中
  with open(md5file, 'wb') as fobj:
    pickle.dump(md5dict, fobj)
##############################增量备份###################################
def incr_backup(src_dir, dst_dir, md5file):
  fname = os.path.basename(src_dir.rstrip('/'))
  fname = '%s_incr_%s.tar.gz' % (fname, time.strftime('%Y%m%d'))
  fname = os.path.join(dst_dir, fname)
  md5dict = {}
  #以二进制的方式读取之前备份的文件
  with open(md5file, 'rb') as fobj:
    oldmd5 = pickle.load(fobj)
  #遍历源目录里的文件，记录每一个md5值，与文件一一对应写入md5dict字典中
  for path, folders, files in os.walk(src_dir):
    for each_file in files:
      key = os.path.join(path, each_file)
      md5dict[key] = check_md5(key)
  #最后将md5dict字典以二进制格式写入新文件中      
  with open(md5file, 'wb') as fobj:
    pickle.dump(md5dict, fobj)
  tar = tarfile.open(fname, 'w:gz')
  #以md5值判断是否相同，如果不同就备份新文件
  for key in md5dict:
    if oldmd5.get(key) != md5dict[key]:
      tar.add(key)
  tar.close()

#如果是周一就完整备份，其他时间增量备份
if __name__ == '__main__':
  # mkdir /tmp/demo; cp -r /etc/security /tmp/demo
  src_dir = '/tmp/demo/security'
  dst_dir = '/var/tmp/backup'  # mkdir /var/tmp/backup
  md5file = '/var/tmp/backup/md5.data'
  if time.strftime('%a') == 'Mon':
    full_backup(src_dir, dst_dir, md5file)
  else:
    incr_backup(src_dir, dst_dir, md5file)
```

