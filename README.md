# EEG-process
clear;clc;
% addpath(genpath('H:\toolbox\eeglab2023.1'));
filepath = 'H:\huangxiaoyang\huangxiaoyang\reject\reject(donation)';
groupfile = dir(fullfile(filepath,'*.set'));
filename = {groupfile.name};
cond = {'S 11','S 12','S 21','S 22'};
for i = 1:length(filename)
    EEG = pop_loadset('filepath',filepath,'filename',filename{1,i});
    EEG = eeg_checkset(EEG);
    for j = 1:length(cond)
        EEG_new = pop_epoch(EEG,cond(j),[-0.2 0.8]);
        EEG_new = pop_rmbase(EEG_new,[-200 0]);
        EEG_avg(i,j,:,:) = squeeze(mean(EEG_new.data,3));
    end
end
time = EEG.times;
time = time';
electro = EEG.chanlocs;
save('N2.mat','time','electro','EEG_avg','cond');
%%
cd('H:\huangxiaoyang\huangxiaoyang\outcomes');
% load('EEG_outcomes.mat');
elec = 32;
displayname = {'Neutral(Emotion) - Positive(Framing)','Neutral(Emotion) - Negative(Framing)','Negative(Emotion)- Positive(Framing)','Negative(Emotion) - Negative(Framing)'};
linecolor = {'-k','-r','--k','--r'};
figure
for i = 1:length(cond)
    plot(time,squeeze(mean(EEG_avg(:,i,elec,:),1)),linecolor{i},'DisplayName',displayname{i},'linewidth',3);hold on;
end
box off;
axis([-200  800 -9 1])
legend('fontname','Times New Roman','fontsize',25,'location','northeast');
legend('boxoff');
set(gca,'ydir','reverse','XAxisLocation','origin','YAxisLocation','origin','fontname','times new roman','fontweight','bold','fontsize',25,'linewidth',3);
xlabel('ms','fontname','times new roman','fontweight','bold','fontsize',30);
ylabel('uV','fontname','times new roman','fontweight','bold','fontsize',30);


%%
latency = find((time>=280)&(time<350));
elec = 32;
for i = 1:length(cond)
    N2_amp(:,i) = squeeze(mean(EEG_avg(:,i,elec,:),1));
    N2_peak(:,i) = min(N2_amp(latency,i));
    N2_peak_lat(:,i) = time(N2_amp(:,i)==N2_peak(i));
    N2_peak_index(:,i) = find((N2_amp(:,i)==N2_peak(i)));
    N2_Stastic(:,i) = squeeze(mean(EEG_avg(:,i,elec,N2_peak_index(:,i)-20:N2_peak_index(:,i)+20),4));
end
xlswrite('N2_Stastic',N2_Stastic,'sheet1');

%%
latency = find((time>=280)&(time<350));
maplimit = [-9,9];
displayname = {'Neutral(Emotion) - Positive(Framing)','Neutral(Emotion) - Negative(Framing)','Negative(Emotion)- Positive(Framing)','Negative(Emotion) - Negative(Framing)'};

figure
for i = 1:length(cond)
    subplot(1,4,i)
    topoplot(squeeze(mean(mean(EEG_avg(:,i,:,N2_peak_index(i)-20:N2_peak_index(i)+20),4),1)),electro,'maplimits',maplimit);
    title(displayname{i},'fontname','Times New Roman','fontsize',17,'fontweight','bold');
end
