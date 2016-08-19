 equire "csv"
require "rake/clean"

datasets = CSV.table "datasets.csv"

def reconstructed_from_raw sample, flat
  file1 = File.basename(sample, ".*")
  file2 = File.basename(flat, ".*")
  dir = File.dirname(sample)
  File.join(dir, "#{file1}_#{file2}.h5")
end

def roi_from_reconstructed reconstructed
  File.join(["data", File.basename(reconstructed, ".h5") + ".roi"])
end

datasets[:reconstructed] = datasets[:sample].zip(
  datasets[:flat]).map {|s, f| reconstructed_from_raw(s, f)}
datasets[:roi] = datasets[:reconstructed].map {|f| roi_from_reconstructed(f)}


namespace :reconstruction do

  datasets.each do |row|
    reconstructed = row[:reconstructed]
    CLEAN.include(reconstructed)

    desc "dpc_reconstruction of #{reconstructed}"
    file reconstructed => [row[:sample], row[:flat]] do |f|
      Dir.chdir "../dpc_reconstruction" do
        sh "dpc_radiography --group /entry/data #{f.prerequisites.join(' ')}"
      end
    end
  end

  desc "csv with reconstructed datasets"
  file "reconstructed.csv" => datasets[:reconstructed] do |f|
    File.open(f.name, "w") do |file|
      file.write(datasets.to_csv)
    end
  end
  CLOBBER.include("reconstructed.csv")

end

task :default => ["reconstructed.csv"]
