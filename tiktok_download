import requests
import re
import json
import os
import sys
import psutil
import time
from concurrent.futures import ThreadPoolExecutor
from multiprocessing import cpu_count; cpu = cpu_count() * 5


class video:
    def __init__(self, url):
        self.url = url
        self.user_topic_music_name = ''
        self.work_finish = 0
        self.headers = {
            'user-agent': 'Mozilla/5.0 (Linux; Android 6.0; Nexus 5 Build/MRA58N) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.106 Mobile Safari/537.36'}
        self.download_headers = {
            'User-Agent': 'Mozilla/5.0 (Android 5.1.1; Mobile; rv:68.0) Gecko/68.0 Firefox/68.0',
        }

    def speed_test(self):
        s1 = psutil.net_io_counters(pernic=True)['WLAN']
        time.sleep(1)
        s2 = psutil.net_io_counters(pernic=True)['WLAN']
        result = s2.bytes_recv - s1.bytes_recv
        # 除法结果保留两位小数
        net_speed = str('%.2f' % (result / 1024 / 1024)) + 'Mb/s'
        return net_speed

    def get_single_work_data(self):
        # 匹配模式
        mode = re.compile(r'http[s]?://(?:[a-zA-Z]|[0-9]|[$-_@.&+]|[!*\(\),]|(?:%[0-9a-fA-F][0-9a-fA-F]))+')
        self.url = re.findall(mode, self.url)[0]
        # 获取访问链接
        rep = requests.get(url=self.url, headers=self.headers, timeout=5).url
        # print(rep)
        r = 'video[/](.*?)[/]'
        item_id = re.findall(r, rep)[0]
        ret_url = 'https://www.iesdouyin.com/web/api/v2/aweme/iteminfo/?item_ids={}&dytk='.format(item_id)
        response = requests.get(url=ret_url, headers=self.headers, timeout=5).text
        response_json = json.loads(response)
        video_data = {}
        # item_list[0].video.vid
        vid = response_json['item_list'][0]['video']['vid']
        url = 'https://aweme.snssdk.com/aweme/v1/play/?video_id={}&line=0&ratio=720p&media_type=4&vr_type=0&improve_bitrate=0&is_play_url=1&is_support_h265=0&source=PackSourceEnum_PUBLISH'.format(vid)
        # item_list[0].desc
        name = response_json['item_list'][0]['desc']
        video_data[name] = url
        self.user_topic_music_name = 'Single download'
        return video_data

    def get_user_allworks_data(self):
        """get the user's basic info
           get username sec_uid favourite/following list
           get video's name and url"""

        # 去掉链接两边空格
        self.url = self.url.strip()
        rep = requests.get(url=self.url, headers=self.headers, timeout=5).url
        # print(rep)               # 获得用户信息返回地址
        r = 'sec_uid=(.*?)&timestamp'
        sec_uid = re.findall(r, rep)
        # print(sec_uid)                   # 获取用户第二名称sec_uid

        # 构造用户信息链接
        get_info_url = 'https://www.iesdouyin.com/web/api/v2/user/info/?sec_uid={}'.format(sec_uid[0])
        # print(get_info_url)
        response = requests.get(url=get_info_url, headers=self.headers, timeout=5).text
        # print(response)
        response_json = json.loads(response)
        # print(response_json)

        # user_info.favoriting_count
        # 获取用户作品列表最大数量
        video_coun = response_json['user_info']['aweme_count']
        # print(video_coun)
        # 获取列表喜欢数量
        fav_coun = response_json['user_info']['favoriting_count']  # 获取列表喜欢数量
        self.user_topic_music_name = response_json['user_info']['nickname']
        max_cursor = [0]
        video_data = {}
        video_data_len = len(video_data)

        print('正在处理用户信息请稍后...', end=' ')
        try:
            for i in range(1024):
                # count = 0
                # print(len(video_data))
                # print(max_cursor[i])
                zhi_url = 'https://www.iesdouyin.com/web/api/v2/aweme/post/?sec_uid={}&count=21&max_cursor={}&aid=1128&_signature=IOprlgAAf.PIMEnreVOKGiDqa4&dytk='.format(
                    sec_uid[0], max_cursor[i])
                data_com = {'status_code': 0, 'has_more': True, 'aweme_list': []}
                response = requests.get(url=zhi_url, headers=self.headers, timeout=10)
                data = json.loads(response.text)
                # print(data) 一般为空
                # print("程序正在处理第{}页信息,请稍后...".format(i + 1))
                # 暴力循环，获取json
                while data == data_com:
                    response = requests.get(url=zhi_url, headers=self.headers, timeout=10)
                    data = json.loads(response.text)
                # 获取max_cursor,该值为页面标志
                try:
                    for j in range(20):
                        max_cursor_num = data['max_cursor']
                        # print(data['extra']['now'])
                        url = data['aweme_list'][j]['video']['play_addr']['url_list'][0]
                        name = data['aweme_list'][j]['desc']
                        # print(url)
                        # 抖音吝啬，将画质调低我们再次将其提高
                        url = url.replace('540', '720')
                        # print(url)
                        # aweme_list[""0""].desc
                        # 放进字典中，方便函数间传递
                        video_data[name] = url

                except:
                    pass
                # 当获取到的视频数量到达时，停止
                if video_data_len >= video_coun:
                    # print('yes')
                    break
                # print(max_cursor)

                # 用页面计数，防止私密视频或者其他情况
                if (max_cursor_num in max_cursor) & (i >= 1):
                    break
                max_cursor.append(max_cursor_num)
        except:
            print('没有那么多视频了！')
        return video_data

    def get_topic_allworks_data(self, counter):
        # 匹配模式
        mode = re.compile(r'http[s]?://(?:[a-zA-Z]|[0-9]|[$-_@.&+]|[!*\(\),]|(?:%[0-9a-fA-F][0-9a-fA-F]))+')
        self.url = re.findall(mode, self.url)[0]
        # 获取访问链接
        rep = requests.get(url=self.url, headers=self.headers, timeout=5).url
        r = 'challenge[/](.*?)[/][?]u_code'
        ch_id = re.findall(r, rep)[0]
        # print(ch_id)

        # 获取话题名称
        topic_url = 'https://www.iesdouyin.com/web/api/v2/challenge/info/?ch_id={}'.format(ch_id)
        res = requests.get(url=topic_url, headers=self.headers, timeout=5).text
        res_josn = json.loads(res)
        # ch_info.cha_name
        self.user_topic_music_name = res_josn['ch_info']['cha_name']
        # print(topic_name)

        video_data = {}
        coun = 0
        try:
            print('正在处理，请稍后...')
            for i in range(2048):
                if coun >= counter:
                    break
                # 构造回调链接
                # i记得需要乘9
                ret_url = 'https://www.iesdouyin.com/web/api/v2/challenge/aweme/?ch_id={}&count=9&cursor={}&aid=1128&screen_limit=3&download_click_limit=0&_signature=6mrJzwAAtXQCsOuysLgQlOpqyd'.format(ch_id,
                                                                                                                                                                                                         i*9)
                response = requests.get(url=ret_url, headers=self.headers, timeout=5).text
                # print(response)
                response_json = json.loads(response)
                # aweme_list[1].video.vid
                try:
                    for j in range(10):
                        vid = response_json['aweme_list'][j]['video']['vid']
                        # aweme_list[0].desc
                        # 利用vid构造直链
                        zhi_url = 'https://aweme.snssdk.com/aweme/v1/play/?video_id={}&line=0&ratio=720p&media_type=4&vr_type=0&improve_bitrate=0&is_play_url=1&is_support_h265=0&source=PackSourceEnum_PUBLISH'.format(vid)
                        name = response_json['aweme_list'][j]['desc']
                        video_data[name] = zhi_url
                        coun += 1
                except:
                    pass
        except:
            print('没有那么多视频了！')
        return video_data

    def get_music_allworks_data(self, counter):
        # 匹配模式
        mode = re.compile(r'http[s]?://(?:[a-zA-Z]|[0-9]|[$-_@.&+]|[!*\(\),]|(?:%[0-9a-fA-F][0-9a-fA-F]))+')
        self.url = re.findall(mode, self.url)[0]
        # 获取访问链接
        rep = requests.get(url=self.url, headers=self.headers, timeout=5).url
        r = 'music[/](.*?)[?]'
        music_id = re.findall(r, rep)[0]
        # print(music_id)
        music_url = 'https://www.iesdouyin.com/web/api/v2/music/info/?music_id={}'.format(music_id)
        music_json = requests.get(url=music_url, headers=self.headers, timeout=5).text
        music_json = json.loads(music_json)
        # music_info.title
        music_name = music_json['music_info']['title']
        # print(music_name)
        self.user_topic_music_name = music_name
        video_data = {}
        coun = 0
        try:
            print('正在处理，请稍后...')
            for i in range(1024):
                # 获取直链 记住i要乘9
                if coun >= counter:
                    break
                zhi_url = 'https://www.iesdouyin.com/web/api/v2/music/list/aweme/?music_id={}&count=9&cursor={}&aid=1128&screen_limit=3&download_click_limit=0&_signature=mbCRwgAAxr9xarO.5yzj85mwkd'.format(music_id, i*9)
                response = requests.get(url=zhi_url, headers=self.headers, timeout=5).text
                response_json = json.loads(response)
                try:
                    for j in range(10):
                        # aweme_list[1].video.play_addr.url_list[0]
                        url = response_json['aweme_list'][j]['video']['play_addr']['url_list'][0]
                        url = url.replace('540', '720')
                        # aweme_list[4].desc
                        name = response_json['aweme_list'][j]['desc']
                        video_data[name] = url
                        coun += 1
                except:
                    pass
        except:
            print('没有更多视频了')
        return video_data

    def single_download(self):
        print("程序开始下载视频,请到F:\Single download\中查看")
        os.makedirs('F:/Single download', exist_ok=True)
        video_data = self.get_single_work_data()
        for name, url in video_data.items():
            # print(url)
            # 防止操作系统操作错误
            name = name.replace('\"', '')
            path = 'F:/{}/{}.mp4'.format(self.user_topic_music_name, name)
            response = requests.get(url=url, headers=self.download_headers, timeout=10).content
            with open(path, 'wb') as f:
                f.write(response)
                f.close()
            print("下载完成")

    def download(self, url, name, datalen):
        # count为了防止暴力循环时卡住
        coun = 25
        path = 'F:/{}/{}.mp4'.format(self.user_topic_music_name, name)
        response = requests.get(url=url, headers=self.download_headers, timeout=10).content
        while response == b'':
            if coun > 0:
                response = requests.get(url=url, headers=self.download_headers, timeout=10).content
            else:
                break

        # 多个进度条
        # response = requests.get(url=url, headers=self.download_headers, timeout=10)
        # # print(response.headers)
        # # 获取文件大小
        # total_size = int(int(response.headers["Content-Length"]) / 1024 + 0.5)
        # # print(total_size)

        # 当阻塞时间过长，结束
        done = int(50 * self.work_finish / datalen)
        # 打印进度条
        sys.stdout.write("\r[%s%s] %d%% %s" % ('█' * done, ' ' * (50 - done), 100 * self.work_finish / datalen,
                                               self.speed_test()))
        sys.stdout.flush()
        try:
            with open(path, 'wb') as f:
                # 多个任务进度条
                # print(name)
                # with tqdm(iterable=response.iter_content(1024), total=total_size, unit='k') as t:
                #     for chunk in t:
                #         f.write(chunk)
                #     t.close()
                f.write(response)
                f.close()
                self.work_finish += 1
            # print(done)
        except:
            pass

    def go(self, choice):
        global video_data
        t_pool = ThreadPoolExecutor(max_workers=cpu)
        if choice == '2':
            video_data = self.get_user_allworks_data()
        elif choice == '3':
            video_data = self.get_topic_allworks_data(int(input('输入你想下载的视频数量：')))
        elif choice == '4':
            video_data = self.get_music_allworks_data(int(input('输入你想下载的视频数量：')))
        name_list = []
        url_list = []
        for name, url in video_data.items():
            name_list.append(name)
            url_list.append(url)
        print("共收集到了{}个视频信息".format(len(video_data)))
        print("程序开始下载视频,请到F:\{}\中查看".format(self.user_topic_music_name))
        os.makedirs('F:/{}'.format(self.user_topic_music_name), exist_ok=True)


        # 计算时间
        self.start_time = time.time()

        # 线程池
        for i in range(len(video_data)):
            t_pool.submit(self.download, url_list[i], name_list[i], len(video_data))
        t_pool.shutdown()
        if self.work_finish / len(video_data) >= 0.98:
            sys.stdout.write("\r[%s%s] %d%% %s 下载完成！" % ('█' * 50, ' ' * 0, 100, '0Mb/s'))
            sys.stdout.flush()


if __name__ == '__main__':
    while 1:
        print(' ----------------------------------------------------------')
        print('|1.下载单个用户视频               |  2.下载某用户所有作品     |')
        print('|3.下载某挑战或话题的指定数量视频   |  4.下载某音乐指定数量视频 |')
        print('|5.退出                          |                         | ')
        print(' ----------------------------------------------------------')
        choice = input('输入对应数字：')
        if choice == '5':
            sys.exit()
        if choice == '1':
            try:
                url = input(("输入分享链接:"))
                video(url).single_download()
                print('\n')
                continue
            except Exception as e:
                print("输入有误,", e, "\n")
                continue
        if choice == '2' or choice == '3' or choice == '4':
            try:
                url = input(("输入分享链接:"))
                video(url).go(choice)
                print('\n')
                continue
            except Exception as e:
                print("输入有误,", e, '\n')
                continue
        else:
            print('输入有误')
    os.system("pause")
    # video(url=input('输入：')).go('2')
